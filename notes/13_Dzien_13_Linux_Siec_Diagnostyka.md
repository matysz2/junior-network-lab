# Dzień 13 --- Linux: sieć i diagnostyka serwera

## Cel dnia

Poznanie podstawowych narzędzi diagnostycznych Linuxa używanych przez
administratora sieci. Ćwiczenia wykonano na rzeczywistym serwerze
Linux/Ubuntu.

### Środowisko laboratorium

-   Serwer: `192.168.0.51/24`
-   Interfejs LAN: `eno1`
-   Brama: `192.168.0.1`
-   Klient Windows: `192.168.0.99`
-   Dostęp administracyjny: SSH, TCP/22

------------------------------------------------------------------------

## 1. Logowanie do serwera przez SSH

Na Windows:

``` powershell
ssh mateusz-czarnik@192.168.0.51
```

Schemat:

``` text
ssh użytkownik@adres_IP_serwera
```

SSH domyślnie korzysta z portu TCP 22. Podczas wpisywania hasła znaki
nie są wyświetlane.

------------------------------------------------------------------------

## 2. Sprawdzenie interfejsów i adresów IP

``` bash
ip -br address
```

Najważniejszy wynik:

``` text
eno1  UP  192.168.0.51/24
```

Znaczenie:

-   `eno1` --- fizyczny interfejs Ethernet,
-   `UP` --- interfejs działa,
-   `192.168.0.51` --- IPv4 serwera,
-   `/24` --- maska `255.255.255.0`.

Pozostałe interfejsy:

-   `lo` --- loopback,
-   `tailscale0` --- interfejs Tailscale,
-   `docker0` --- podstawowa sieć Dockera,
-   `br-*` --- mosty sieciowe Dockera,
-   `veth*` --- wirtualne interfejsy kontenerów.

------------------------------------------------------------------------

## 3. Tablica routingu

``` bash
ip route
```

Najważniejszy wpis:

``` text
default via 192.168.0.1 dev eno1 proto dhcp src 192.168.0.51 metric 100
```

Znaczenie:

-   `default` --- trasa używana, gdy nie istnieje bardziej szczegółowa,
-   `via 192.168.0.1` --- brama / next hop,
-   `dev eno1` --- interfejs wyjściowy,
-   `src 192.168.0.51` --- adres źródłowy.

Sieć lokalna:

``` text
192.168.0.0/24 dev eno1
```

Serwer jest do niej podłączony bezpośrednio.

Trasy `172.17.0.0/16`, `172.18.0.0/16` i `172.19.0.0/16` dotyczą sieci
Dockera.

------------------------------------------------------------------------

## 4. Sprawdzenie wyboru trasy do Internetu

``` bash
ip route get 8.8.8.8
```

Wynik:

``` text
8.8.8.8 via 192.168.0.1 dev eno1 src 192.168.0.51
```

Czyli:

``` text
serwer 192.168.0.51
        |
       eno1
        |
        v
brama 192.168.0.1
        |
        v
      Internet
```

------------------------------------------------------------------------

## 5. Trasa do urządzenia w tej samej podsieci

``` bash
ip route get 192.168.0.100
```

Wynik:

``` text
192.168.0.100 dev eno1 src 192.168.0.51
```

Nie występuje `via 192.168.0.1`, ponieważ `192.168.0.51/24` i
`192.168.0.100/24` znajdują się w tej samej podsieci.

------------------------------------------------------------------------

## 6. Tablica sąsiadów --- IP i MAC

``` bash
ip neigh
```

Dla bramy:

``` text
192.168.0.1 dev eno1 lladdr 58:d5:6e:c3:8d:09 REACHABLE
```

-   IP bramy: `192.168.0.1`
-   MAC bramy: `58:d5:6e:c3:8d:09`
-   `REACHABLE` --- sąsiad został potwierdzony jako osiągalny.

Routing wskazuje adres IP następnego urządzenia, a mechanizm
neighbour/ARP pozwala ustalić jego MAC w lokalnym Ethernecie.

------------------------------------------------------------------------

## 7. Test bramy

``` bash
ping -c 4 192.168.0.1
```

Wynik laboratorium:

``` text
4 packets transmitted, 4 received, 0% packet loss
```

Brama jest osiągalna.

------------------------------------------------------------------------

## 8. Test Internetu bez DNS

``` bash
ping -c 4 8.8.8.8
```

Wynik:

``` text
4 packets transmitted, 4 received, 0% packet loss
```

Test adresu IP pozwala oddzielić problem z routingiem/Internetem od
problemu z DNS.

------------------------------------------------------------------------

## 9. Test DNS

``` bash
resolvectl query example.com
```

DNS poprawnie zwrócił adresy IPv4 i IPv6 dla `example.com`.

Jeżeli:

``` text
ping 8.8.8.8 -> działa
nazwa domenowa -> nie działa
```

jednym z pierwszych podejrzeń jest DNS.

------------------------------------------------------------------------

## 10. Sprawdzenie konfiguracji DNS

``` bash
resolvectl status
```

Dla `eno1`:

``` text
Current DNS Server: 192.168.0.1
DNS Servers: 192.168.0.1 ...
```

Dla Tailscale:

``` text
Current DNS Server: 100.100.100.100
```

Serwer korzysta z `systemd-resolved`. Z widoku oznaczonego `(END)`
wychodzimy klawiszem:

``` text
q
```

------------------------------------------------------------------------

## 11. Porty nasłuchujące

``` bash
sudo ss -tulpn
```

Opcje:

-   `-t` --- TCP,
-   `-u` --- UDP,
-   `-l` --- tylko nasłuchujące,
-   `-p` --- proces,
-   `-n` --- numery portów.

W laboratorium potwierdziliśmy m.in.:

-   SSH --- TCP/22,
-   Samba --- TCP/445 i TCP/139,
-   usługi wystawione przez Docker.

------------------------------------------------------------------------

## 12. Stan usługi SSH

``` bash
systemctl status ssh
```

Wynik:

``` text
Active: active (running)
```

oraz:

``` text
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
```

`enabled` oznacza skonfigurowanie uruchamiania usługi przy starcie
systemu.

------------------------------------------------------------------------

## 13. Logi SSH

``` bash
sudo journalctl -u ssh -n 20
```

Znaczenie:

-   `journalctl` --- logi systemowe,
-   `-u ssh` --- tylko usługa SSH,
-   `-n 20` --- ostatnie 20 wpisów.

W logach widzieliśmy m.in. uruchamianie SSH, nasłuchiwanie na porcie 22
i udane logowania.

------------------------------------------------------------------------

## 14. Filtrowanie wyników za pomocą potoku

``` bash
sudo ss -ltnp | grep ':22'
```

Znak:

``` text
|
```

to **pipe / potok**. Wynik pierwszego polecenia jest przekazywany do
następnego.

Uwaga: `grep ':22'` znalazł również port `2283`, ponieważ tekst `:2283`
także zaczyna się od `:22`.

------------------------------------------------------------------------

## 15. Test portu SSH z Windows

W PowerShell:

``` powershell
Test-NetConnection 192.168.0.51 -Port 22
```

Wynik:

``` text
SourceAddress    : 192.168.0.99
RemoteAddress    : 192.168.0.51
RemotePort       : 22
TcpTestSucceeded : True
```

To potwierdziło, że klient Windows może nawiązać połączenie TCP z portem
22 serwera.

Ping i test portu nie oznaczają tego samego. Ping może działać, mimo że
konkretna usługa jest niedostępna.

------------------------------------------------------------------------

## 16. Firewall UFW

``` bash
sudo ufw status verbose
```

Wynik laboratorium:

``` text
Status: inactive
```

Nie włączaliśmy UFW podczas zdalnej sesji SSH. Przed zmianą reguł
firewalla na zdalnym serwerze należy zachować możliwość dostępu i
cofnięcia zmian.

------------------------------------------------------------------------

## 17. Diagnostyka problemu z SSH

Jeżeli użytkownik zgłasza, że SSH nie działa:

``` text
1. Czy serwer ma poprawny IP?
        |
2. Czy serwer jest osiągalny?
        |
3. Czy TCP/22 jest osiągalny?
        |
4. Czy port 22 nasłuchuje?
        |
5. Czy ssh.service działa?
        |
6. Czy firewall blokuje ruch?
        |
7. Co pokazują logi?
```

Przydatne polecenia:

``` bash
ip -br address
ip route
sudo ss -ltnp
systemctl status ssh
sudo ufw status verbose
sudo journalctl -u ssh -n 20
```

Jeżeli SSH nie działa, nie zakładamy, że naprawimy je przez tę samą
niedziałającą sesję SSH. Potrzebujemy np. lokalnej konsoli serwera,
konsoli maszyny wirtualnej lub innej ścieżki administracyjnej.

------------------------------------------------------------------------

## 18. Uniwersalny schemat diagnostyki sieci

``` text
1. INTERFEJS + IP
        |
2. ROUTING + BRAMA
        |
3. PING BRAMY
        |
4. INTERNET PO IP
        |
5. DNS
        |
6. KONKRETNY PORT
        |
7. USŁUGA
        |
8. FIREWALL
        |
9. LOGI
```

### Najważniejsza zasada

Nie zmieniamy przypadkowo konfiguracji. Najpierw:

``` text
objaw -> hipoteza -> test -> interpretacja -> naprawa -> weryfikacja
```

Celem administratora nie jest zapamiętanie wszystkich poleceń, ale
wiedza **co sprawdzić, w jakiej kolejności i jak zinterpretować wynik**.
