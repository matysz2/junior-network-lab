# Dzień 14 - DNS i DHCP w izolowanym laboratorium Linux

## Cel

Praktyczne ćwiczenie DHCP, DNS, DORA, portów, `network namespace`,
`veth` oraz diagnostyki: **objaw → hipoteza → test → interpretacja →
naprawa → weryfikacja**.

## 1. Sprawdzenie stanu

``` bash
dpkg -l | grep -E 'dnsmasq|isc-dhcp-server|bind9'
sudo ss -tulpn | grep -E ':53 |:67 '
ip link help | head
ip netns help
```

Port 53 był lokalnie używany przez `systemd-resolved` na
`127.0.0.53/127.0.0.54`. Na porcie UDP/67 nic nie nasłuchiwało.

## 2. Izolowany klient

``` bash
sudo ip netns add klient14
ip netns list
sudo ip netns exec klient14 ip -br address
sudo ip link add veth-serwer type veth peer name veth-klient
sudo ip link set veth-klient netns klient14
sudo ip addr add 10.14.0.1/24 dev veth-serwer
sudo ip link set veth-serwer up
sudo ip netns exec klient14 ip link set veth-klient up
```

Schemat:

``` text
Ubuntu host
veth-serwer 10.14.0.1/24
      ||
      || veth
      ||
veth-klient
klient14
```

W trakcie ćwiczenia wystąpiły i zostały zdiagnozowane błędy
`Invalid "netns" value` oraz `Cannot find device "veth-klient"` -
brakowało odpowiednio namespace i interfejsu po odtworzeniu sesji.

## 3. DHCP

Plik `/tmp/dnsmasq-lab.conf`:

``` ini
interface=veth-serwer
bind-interfaces
port=0
dhcp-range=10.14.0.100,10.14.0.150,255.255.255.0,1h
dhcp-option=3,10.14.0.1
dhcp-option=6,10.14.0.1
```

Test i uruchomienie:

``` bash
sudo dnsmasq --test --conf-file=/tmp/dnsmasq-lab.conf
sudo dnsmasq --conf-file=/tmp/dnsmasq-lab.conf --no-daemon
sudo apt install isc-dhcp-client
which dhclient
sudo ip netns exec klient14 dhclient -v veth-klient
```

Klient dostał `10.14.0.147/24`.

### DORA

-   **Discover** - klient rozgłasza poszukiwanie serwera DHCP.
-   **Offer** - serwer oferuje adres i parametry.
-   **Request** - klient prosi o przydzielenie wybranego adresu.
-   **Acknowledge** - serwer zatwierdza dzierżawę.

``` text
DHCPDISCOVER
DHCPOFFER   10.14.0.147
DHCPREQUEST 10.14.0.147
DHCPACK     10.14.0.147
```

Weryfikacja:

``` bash
sudo ip netns exec klient14 ip -br address
sudo ip netns exec klient14 ip route
sudo ip netns exec klient14 ping -c 4 10.14.0.1
```

## 4. DNS

Plik `/tmp/dnsmasq-dns-lab.conf`:

``` ini
interface=veth-serwer
bind-interfaces
port=53
no-resolv
address=/serwer.lab/10.14.0.1
```

Podczas pierwszej próby `dnsmasq --test` wykrył błąd
`Bad address in --address at line 5`. Polecenie:

``` bash
cat -n /tmp/dnsmasq-dns-lab.conf
```

ujawniło sklejoną linię i zdublowaną konfigurację. Po poprawieniu pliku:

``` bash
sudo dnsmasq --test --conf-file=/tmp/dnsmasq-dns-lab.conf
sudo dnsmasq --conf-file=/tmp/dnsmasq-dns-lab.conf --no-daemon
sudo ip netns exec klient14 dig @10.14.0.1 serwer.lab
```

Odpowiedź:

``` text
serwer.lab.  0  IN  A  10.14.0.1
status: NOERROR
SERVER: 10.14.0.1#53 (UDP)
```

## 5. Kontrolowana awaria DNS

Po `Ctrl+C`:

``` bash
sudo ip netns exec klient14 dig @10.14.0.1 serwer.lab
```

zwróciło `connection refused`.

Test:

``` bash
sudo ss -tulpn | grep ':53 '
```

pokazał tylko `systemd-resolved` na `127.0.0.53/127.0.0.54`, bez
nasłuchu `10.14.0.1:53`.

Naprawa:

``` bash
sudo dnsmasq --conf-file=/tmp/dnsmasq-dns-lab.conf --no-daemon
```

Ponowny `dig` zwrócił `NOERROR` i `10.14.0.1`.

## 6. Porty

  Usługa        Port
  ------------- ------------
  SSH           TCP 22
  DNS           UDP/TCP 53
  DHCP serwer   UDP 67
  DHCP klient   UDP 68
  HTTP          TCP 80
  HTTPS         TCP 443

## 7. Szybka diagnostyka

1.  Brak IP → DHCP / interfejs / VLAN / kabel.
2.  Jest IP, brak bramy → lokalna konfiguracja, ARP/neighbor, VLAN,
    firewall.
3.  Brama działa, ale nie działa zewnętrzny IP → routing/łącze dalej.
4.  Zewnętrzny IP działa, ale nazwy nie → DNS.

**Do powtórki:** DNS = port 53. `connection refused` na `#53` → sprawdź
usługę i nasłuch DNS.

## 8. Cleanup

``` bash
sudo ip netns delete klient14
ip netns list
sudo ip link delete veth-serwer
ip -br address show veth-serwer
sudo rm /tmp/dnsmasq-lab.conf /tmp/dnsmasq-dns-lab.conf
sudo rm /tmp/dnsmasq-dns-lab.conf.save
ls /tmp/dnsmasq*
```

Zweryfikowano: `Device "veth-serwer" does not exist.`

## Wynik dnia

Wykonano i zweryfikowano: izolację namespace/veth, DHCP i DORA,
przydział `10.14.0.147/24`, routing lokalny, DNS
`serwer.lab → 10.14.0.1`, kontrolowaną awarię DNS, diagnostykę i naprawę
oraz cleanup.
