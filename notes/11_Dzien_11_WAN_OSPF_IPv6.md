# Dzień 11 --- WAN, OSPF i podstawy IPv6

## Cel dnia

Przećwiczyć połączenie dwóch routerów jako uproszczony WAN, uruchomić
OSPF w jednym obszarze, sprawdzić dynamicznie poznane trasy, wykonać
diagnostykę awarii oraz poznać podstawowe polecenia IPv6.

> Uwaga: program Dnia 11 obejmuje także wstęp do redundancji bramy. W
> tej sesji tego fragmentu jeszcze nie wykonywaliśmy praktycznie, więc
> nie oznaczam go jako ukończony.

## Topologia wykonana w Packet Tracerze

``` text
VLAN 10: 192.168.10.0/24
VLAN 20: 192.168.20.0/24
          |
       Router1
 G0/1 10.0.0.1/30
          |
       WAN / OSPF
          |
 G0/0 10.0.0.2/30
       Router2
 G0/1 192.168.30.1/24
          |
       Switch2
          |
 PC3 192.168.30.10/24
 GW  192.168.30.1
```

## 1. WAN --- najważniejsza idea

LAN to sieć lokalna. WAN łączy sieci/lokalizacje oddalone od siebie. W
laboratorium kabel Router1--Router2 symuluje łącze WAN. W realnej firmie
połączenie może być dostarczone przez operatora albo zrealizowane np.
przez VPN site-to-site.

Łącze między routerami: - Router1 G0/1: `10.0.0.1/30` - Router2 G0/0:
`10.0.0.2/30` - sieć: `10.0.0.0/30`

Test `ping 10.0.0.1` z Router2 zakończył się 4/5 przy pierwszej próbie;
kolejne odpowiedzi działały. Pierwsza utrata mogła wynikać z
początkowego ustalenia informacji sąsiedzkiej/ARP w symulacji.

## 2. Tablica routingu

Router podejmuje decyzję, którędy wysłać pakiet, na podstawie tablicy
routingu.

Najważniejsze oznaczenia: - `C` --- Connected, sieć bezpośrednio
podłączona. - `L` --- Local, adres należący do samego routera. - `S` ---
Static, trasa statyczna. - `O` --- trasa poznana przez OSPF.

Polecenie:

``` text
show ip route
```

Przed OSPF Router1 znał własne sieci `192.168.10.0/24`,
`192.168.20.0/24` i WAN `10.0.0.0/30`, ale nie znał `192.168.30.0/24`.

## 3. OSPF --- po co?

OSPF jest protokołem routingu dynamicznego. Routery mogą wymieniać
informacje o osiągalnych sieciach, zamiast wymagać ręcznego wpisywania
każdej trasy statycznej.

Uruchomienie procesu:

``` text
router ospf 1
```

W laboratorium użyliśmy jednego obszaru:

``` text
area 0
```

### Router1

``` text
router ospf 1
network 10.0.0.0 0.0.0.3 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
```

### Router2

``` text
router ospf 1
network 10.0.0.0 0.0.0.3 area 0
network 192.168.30.0 0.0.0.255 area 0
```

Wildcard: - `/30`, maska `255.255.255.252` -\> `0.0.0.3` - `/24`, maska
`255.255.255.0` -\> `0.0.0.255`

Komenda `network` w OSPF dopasowuje interfejsy, na których OSPF ma
działać, i przypisuje je do obszaru.

## 4. Sąsiedztwo OSPF

Sprawdzenie:

``` text
show ip ospf neighbor
```

W laboratorium uzyskaliśmy stan:

``` text
FULL
```

`FULL` oznacza pełne sąsiedztwo OSPF.

Przykładowy rzeczywisty wynik z ćwiczenia:

``` text
Neighbor ID     Pri   State      Address    Interface
192.168.30.1      1   FULL/DR    10.0.0.2   GigabitEthernet0/1
```

`Neighbor ID` to Router ID OSPF, a `Address` to adres sąsiada na
wspólnym łączu.

## 5. Trasy poznane przez OSPF

Na Router1 pojawiło się:

``` text
O 192.168.30.0/24 [110/2] via 10.0.0.2, GigabitEthernet0/1
```

Czytamy: - `O` --- trasa z OSPF, - `192.168.30.0/24` --- sieć
docelowa, - `via 10.0.0.2` --- next hop, czyli Router2, - `G0/1` ---
interfejs wyjściowy.

Na Router2 pojawiły się:

``` text
O 192.168.10.0/24 ... via 10.0.0.1
O 192.168.20.0/24 ... via 10.0.0.1
```

## 6. Test komunikacji między oddziałami

PC3 w sieci `192.168.30.0/24`: - IP: `192.168.30.10` - maska:
`255.255.255.0` - brama: `192.168.30.1`

Ping do Router1 `192.168.10.1`: - 4 wysłane, - 4 odebrane, - 0% strat.

## 7. Diagnostyka --- brak bramy na PC0

Objaw: PC3 mógł pingować Router1 `192.168.10.1`, ale nie PC0
`192.168.10.10`.

Test na PC0:

``` text
ipconfig
```

Wynik:

``` text
IPv4 Address:    192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 0.0.0.0
```

Hipoteza: PC0 nie ma bramy do odpowiedzi do innej podsieci.

Naprawa:

``` text
Default Gateway: 192.168.10.1
```

Weryfikacja z PC3:

``` text
ping 192.168.10.10
```

Wynik po naprawie: - 4 wysłane, - 4 odebrane, - 0% strat.

Wniosek: jeżeli host komunikuje się lokalnie, ale nie z innymi
podsieciami, jedną z pierwszych rzeczy do sprawdzenia jest brama
domyślna.

## 8. IPv6 --- podstawy

IPv4 ma 32 bity, IPv6 ma 128 bitów.

Przykład IPv6:

``` text
2001:db8:12::1/64
```

`::` skraca ciąg grup zer.

Włączenie routingu IPv6:

``` text
ipv6 unicast-routing
```

Adresy na łączu:

``` text
Router1 G0/1: 2001:db8:12::1/64
Router2 G0/0: 2001:db8:12::2/64
```

To samo łącze miało jednocześnie IPv4 i IPv6 --- przykład dual stack.

Sprawdzenie interfejsów:

``` text
show ipv6 interface brief
```

Na Router1 pojawił się także automatyczny adres `FE80::...`, czyli
link-local.

Test:

``` text
ping 2001:db8:12::1
```

Wynik z Router2: 5/5, 100%.

Tablica IPv6:

``` text
show ipv6 route
```

Na Router2:

``` text
C 2001:DB8:12::/64
L 2001:DB8:12::2/128
```

`C` = Connected, `L` = Local.

## 9. Test awarii OSPF

Na Router1 celowo wyłączono G0/1:

``` text
interface gigabitEthernet0/1
shutdown
```

Skutek:

``` text
OSPF: FULL -> DOWN
```

Trasa `O 192.168.30.0/24` zniknęła z `show ip route`.

Naprawa:

``` text
interface gigabitEthernet0/1
no shutdown
```

Po chwili sąsiedztwo przeszło do `FULL`, a trasa:

``` text
O 192.168.30.0/24 ... via 10.0.0.2
```

wróciła automatycznie.

## 10. Schemat diagnostyczny OSPF

``` text
Brak komunikacji między lokalizacjami
        |
show ip ospf neighbor
        |
brak sąsiada?
        |
show ip interface brief
        |
czy łącze jest up/up?
        |
sprawdzenie konfiguracji OSPF / adresacji
        |
naprawa
        |
show ip ospf neighbor -> FULL
show ip route -> trasa wróciła
ping -> test końcowy
```

DNS nie jest pierwszym testem, gdy problemem jest brak sąsiedztwa OSPF.
DNS służy głównie do tłumaczenia nazw na adresy IP.

## 11. Komendy do zapamiętania

``` text
show ip interface brief
show ip route
show ip ospf neighbor
router ospf 1
network <sieć> <wildcard> area 0
ipv6 unicast-routing
ipv6 address <adres>/<prefix>
show ipv6 interface brief
show ipv6 route
ping <adres>
shutdown
no shutdown
```

## 12. Co wymaga powtórki

Do dalszego utrwalania: - DNS vs routing vs OSPF, - `show ip route` vs
`show ip ospf neighbor`, - `show ip route` dla IPv4 vs `show ipv6 route`
dla IPv6, - znaczenie `via` jako next hop, - tryby Cisco: `Router#`,
`Router(config)#`, `Router(config-if)#`, `Router(config-router)#`.

## Status Dnia 11

Wykonane praktycznie: - WAN między Router1 i Router2, - OSPF
single-area, - sąsiedztwo OSPF `FULL`, - dynamiczna wymiana tras, - test
ruchu między sieciami, - diagnoza i naprawa bramy domyślnej, - awaria
łącza i automatyczna odbudowa OSPF, - podstawy IPv6 i test IPv6.

Nie wykonano jeszcze praktycznie: - redundancji bramy (temat
przewidziany w programie Dnia 11).

## Następny krok

Przed Dniem 12 zrobić krótką powtórkę komend OSPF/IPv4/IPv6. Następnie
Dzień 12: WLAN --- SSID, pasma, kanały, zakłócenia, WPA2/WPA3, sieć
gościnna i sprawdzenie działu.
