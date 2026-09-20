# Dzień 2 - LAN WAN Ethernet i modele OSI TCP IP

## Cel

Rozróżnianie podstawowych rodzajów sieci i urządzeń oraz wskazywanie warstwy, od której należy zacząć diagnostykę.

## Najważniejsze pojęcia

| Pojęcie | Znaczenie | Przykład z laboratorium |
|---|---|---|
| LAN | Sieć lokalna w domu lub biurze | Ubuntu `192.168.0.51` i router `192.168.0.1` |
| WLAN | Bezprzewodowa część LAN | Telefon lub laptop po Wi-Fi |
| WAN | Sieć między lokalizacjami, w domu zwykle połączenie routera z operatorem | Ruch routera dalej do Internetu |
| Switch | Łączy urządzenia w tej samej sieci lokalnej, głównie warstwa 2 | Komputer i drukarka w tym samym VLAN-ie |
| Router | Przekazuje ruch między sieciami, głównie warstwa 3 | Brama `192.168.0.1` prowadząca do Internetu |
| Access Point | Udostępnia urządzeniom dostęp do sieci przez Wi-Fi | Telefon połączony z domowym Wi-Fi |

## Ethernet i stan interfejsu

Ethernet to standard przewodowej sieci lokalnej. W laboratorium interfejs `eno1` uzyskał wynik:

```text
<BROADCAST,MULTICAST,UP,LOWER_UP>
```

- `UP` - interfejs jest włączony w systemie.
- `LOWER_UP` - system wykrywa działające połączenie fizyczne, zwykle kabel i aktywny port po drugiej stronie.
- `mtu 1500` - standardowy maksymalny rozmiar pakietu IP w sieci Ethernet.

Switch 100 Mb/s ograniczy transfer między urządzeniami do maksymalnie 100 Mb/s. Jest wtedy wąskim gardłem, nawet jeżeli karty sieciowe urządzeń obsługują 1 Gb/s.

## OSI i TCP IP

| Warstwa OSI | Co obejmuje | Przykład |
|---:|---|---|
| 7 - aplikacji | Usługi używane przez programy | HTTPS, DNS, SSH |
| 4 - transportowa | TCP, UDP i porty | TCP port 443 dla HTTPS |
| 3 - sieciowa | Adresy IP i routing | Router, IPv4, brama domyślna |
| 2 - łącza danych | Ethernet, MAC, VLAN | Switch, ramka Ethernet |
| 1 - fizyczna | Sygnał i medium | Kabel, wtyk RJ-45, port Ethernet, Wi-Fi |

Model TCP/IP grupuje warstwy OSI: aplikacji (5-7), transportową (4), Internetu (3) i dostępu do sieci (1-2).

## Wykonane ćwiczenie

W Ubuntu przez SSH wykonano:

```bash
ip link show dev eno1
```

Wynik potwierdził `UP` oraz `LOWER_UP`, czyli aktywny interfejs i wykryte połączenie fizyczne.

## Jeżeli nie działa - kabel lub port Ethernet

| Objaw | Hipoteza | Pierwszy test | Prawidłowy wynik | Następny krok |
|---|---|---|---|---|
| Brak sieci po kablu | Kabel wypięty lub uszkodzony | Sprawdź kabel, port i diody | Kabel jest pewnie wpięty, dioda link świeci | Sprawdź `ip link show dev eno1` |
| Brak `LOWER_UP` | Brak fizycznego linku | `ip link show dev eno1` | Widoczne `UP,LOWER_UP` | Zmień kabel lub port, potem powtórz test |
| Jest `LOWER_UP`, ale nie działa Internet | Problem wyżej niż warstwa fizyczna | `ip -br addr`, potem ping bramy | Adres IP i odpowiedź bramy | Sprawdź bramę, DNS albo usługę zależnie od objawu |

## Sprawdzenie wiedzy

- Router działa głównie w warstwie 3 OSI.
- Switch działa głównie w warstwie 2 OSI.
- Wypięty kabel jest problemem warstwy 1 OSI.
- Do komunikacji między `192.168.10.20/24` i `192.168.20.10/24` potrzebny jest router, ponieważ są to różne sieci IP.

## Stan nauki do wznowienia

- Ukończono dzień 2.
- Powtórzyć: warstwy 1, 2 i 3 oraz zadania switcha i routera.
- Następny krok: dzień 3 - IPv4, maska, brama, adresy prywatne i publiczne oraz CIDR.
