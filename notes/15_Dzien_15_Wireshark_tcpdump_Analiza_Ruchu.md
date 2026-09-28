# Dzień 15 --- Wireshark, tcpdump i analiza ruchu

**Kurs:** Junior Network Engineer\
**Data:** 28.09.2026\
**Środowisko:** Ubuntu 24.04.5 LTS, Wireshark/TShark 4.2.2\
**Status:** praktyka Dnia 15 wykonana; kluczowe pojęcia wymagają
krótkiej powtórki.

## Cel

Nauczyć się patrzeć na ruch sieciowy jak administrator: ustalić **kto
wysyła, do kogo, jakim protokołem i na jaki port**, a następnie na
podstawie pakietów zawężać przyczynę problemu.

## Laboratorium

``` text
Sieć domowa:
Ubuntu 192.168.0.51 ---- eno1 ---- router 192.168.0.1 ---- Internet

Izolowany lab DHCP:
HOST Ubuntu                         namespace klient15
10.15.0.1/24
veth15-serwer ===================== veth15-klient
          wirtualny kabel Ethernet
```

Klient DHCP otrzymał rzeczywiście adres **10.15.0.137/24**.

## 1. ICMP i ping

``` bash
ping -c 2 192.168.0.1
sudo tcpdump -ni eno1 icmp
```

Zaobserwowaliśmy: - `192.168.0.51 -> 192.168.0.1` --- Echo Request, -
`192.168.0.1 -> 192.168.0.51` --- Echo Reply.

Do zapamiętania:

``` text
ICMP type 8 = Echo Request
ICMP type 0 = Echo Reply
```

Ping testuje ICMP. Poprawny ping **nie oznacza automatycznie**, że np.
SSH działa.

## 2. PCAP

``` bash
sudo tcpdump -ni eno1 -c 4 -w ~/dzien15_icmp.pcap icmp
sudo tcpdump -nn -r ~/dzien15_icmp.pcap
tshark -r ~/dzien15_icmp.pcap
```

PCAP to zapis przechwyconych pakietów do późniejszej analizy.

## 3. ARP

ARP odpowiada na pytanie: **znam IPv4 urządzenia w lokalnym segmencie
--- jaki jest jego MAC?**

``` bash
ip neigh show 192.168.0.1
sudo ip neigh del 192.168.0.1 dev eno1
sudo tcpdump -ni eno1 arp
```

Zaobserwowaliśmy:

``` text
who-has 192.168.0.1 tell 192.168.0.51
192.168.0.1 is-at 58:d5:6e:c3:8d:09
```

## 4. DNS

``` bash
dig example.com
sudo tcpdump -ni eno1 port 53
```

Rzeczywisty przechwyt:

``` text
192.168.0.51.50651 > 192.168.0.1.53: A? example.com
192.168.0.1.53 > 192.168.0.51.50651: odpowiedź DNS
```

`53` to **port DNS**, a nie numer VLAN-u.

## 5. DHCP --- DORA

Najważniejsza powtórka:

``` text
KLIENT                         SERWER DHCP
  |--- DISCOVER ------------->|  Szukam DHCP
  |<-- OFFER -----------------|  Oferuję adres
  |--- REQUEST -------------->|  Chcę ten adres
  |<-- ACK -------------------|  Potwierdzam
```

**DORA = Discover → Offer → Request → ACK**

Kierunek:

**klient → serwer → klient → serwer**

Porty:

-   serwer DHCP --- UDP 67,
-   klient DHCP --- UDP 68.

W naszym labie:

``` bash
sudo ip netns exec klient15 dhclient -v veth15-klient
sudo ip netns exec klient15 ip -br address
sudo tcpdump -ni veth15-serwer -vv 'udp port 67 or udp port 68'
```

Przechwyciliśmy pełne DORA. Klient dostał **10.15.0.137/24**, bramę
`10.15.0.1` i dzierżawę 3600 s.

## 6. TCP --- 3-way handshake

``` text
Klient                     Serwer
  |--- SYN --------------->|
  |<-- SYN + ACK ----------|
  |--- ACK --------------->|
       połączenie gotowe
```

W tcpdump:

-   `[S]` = SYN,
-   `[S.]` = SYN + ACK,
-   `[.]` = ACK,
-   `[P.]` = PSH + ACK, zwykle są dane,
-   `[F.]` = FIN + ACK, zamykanie.

W naszym przechwycie po handshake klient wysłał `HTTP HEAD`, a serwer
odpowiedział `HTTP/1.1 200 OK`.

## 7. Filtry TShark

``` bash
tshark -r ~/dzien15_icmp.pcap -Y "icmp.type == 8"
tshark -r ~/dzien15_icmp.pcap -Y "icmp.type == 0"
```

U nas: - type 8 pokazał pakiety 1 i 3, - type 0 pokazał pakiety 2 i 4.

**Capture filter** decyduje, co przechwytujemy.\
**Display filter** decyduje, co pokazujemy z już przechwyconych
pakietów.

## 8. Najważniejsze porty

  Port         Usługa
  ------------ -------------
  TCP 22       SSH
  UDP/TCP 53   DNS
  UDP 67       DHCP serwer
  UDP 68       DHCP klient
  TCP 80       HTTP
  TCP 443      HTTPS

## 9. Diagnostyka

Zasada:

``` text
objaw
  ↓
hipoteza
  ↓
jeden test
  ↓
interpretacja
  ↓
najmniejsza potrzebna naprawa
  ↓
ponowny test
```

Przykład: jeśli widzisz SYN wychodzący od klienta, ale nie widzisz
SYN/ACK, wiesz, że połączenie TCP nie zostało zestawione. Nie wiesz
jeszcze dlaczego. Kolejno sprawdzasz usługę/port, firewall, routing i
drogę powrotną.

## 10. Co wymaga powtórki

Na podstawie rzeczywistych odpowiedzi z lekcji:

1.  DORA i role klient/serwer.
2.  ICMP: type 8 = Request, type 0 = Reply.
3.  `[S.]` = SYN + ACK.
4.  Port 53 = DNS.
5.  Nie wyciągać z jednego pakietu zbyt daleko idących wniosków.

## Ściągawka

``` text
ICMP: 8=Request, 0=Reply

DHCP:
Discover -> Offer -> Request -> ACK
klient   <- serwer -> klient <- serwer
(poprawny kierunek strzałek:
klient->serwer, serwer->klient, klient->serwer, serwer->klient)

TCP:
SYN -> SYN/ACK -> ACK

Porty:
22 SSH
53 DNS
67/68 DHCP
80 HTTP
443 HTTPS
```

## Stan nauki do wznowienia

Dzień 15: praktyka wykonana. Przećwiczone: `tcpdump`, ICMP, ARP, DNS,
izolowane DHCP/DORA, TCP handshake, zapis PCAP, Wireshark/TShark i
filtry. Do utrwalenia: DORA, ICMP 8/0, `[S.]`, port 53.

**Następny krok:** krótka poprawka z tych czterech elementów, a
następnie Dzień 16 --- NAT/PAT, ACL i podstawy `nftables`.
