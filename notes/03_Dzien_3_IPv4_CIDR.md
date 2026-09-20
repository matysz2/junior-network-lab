# Dzień 3 - IPv4 maska brama i CIDR

## Cel

Rozpoznawanie adresu IPv4, maski, bramy, sieci prywatnej oraz wyznaczanie adresu sieci i broadcast dla `/24`.

## Adres Ubuntu z laboratorium

| Element | Wartość |
|---|---|
| Adres IP | `192.168.0.51` |
| Prefiks CIDR | `/24` |
| Maska dziesiętna | `255.255.255.0` |
| Adres sieci | `192.168.0.0` |
| Broadcast | `192.168.0.255` |
| Brama domyślna | `192.168.0.1` |

Adres IPv4 ma 32 bity, zapisane jako cztery liczby od 0 do 255. Prefiks `/24` oznacza, że pierwsze 24 bity wskazują sieć. W praktyce dla `/24` pierwsze trzy liczby wskazują sieć, a ostatnia adres urządzenia.

## CIDR

| Prefiks | Maska | Adresy w podsieci | Typowe adresy urządzeń |
|---:|---|---:|---:|
| `/24` | `255.255.255.0` | 256 | 254 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/16` | `255.255.0.0` | 65 536 | 65 534 |

W typowej podsieci dwa adresy są zarezerwowane: adres sieci i broadcast.

## Prywatne zakresy IPv4

| Zakres | Przykład |
|---|---|
| `10.0.0.0/8` | `10.20.30.40` |
| `172.16.0.0/12` | `172.20.5.10` |
| `192.168.0.0/16` | `192.168.0.51` |

Adres Ubuntu `192.168.0.51` jest prywatny. Router zwykle wykonuje NAT, aby urządzenia z prywatnymi adresami mogły korzystać z połączenia z Internetem.

## Test wyboru trasy

Wykonano w Ubuntu:

```bash
ip route get 192.168.0.20
ip route get 1.1.1.1
```

Wyniki:

```text
192.168.0.20 dev eno1 src 192.168.0.51
1.1.1.1 via 192.168.0.1 dev eno1 src 192.168.0.51
```

`192.168.0.20` jest w tej samej sieci `/24`, dlatego serwer wysyła ruch bezpośrednio przez `eno1`. `1.1.1.1` jest poza siecią lokalną, więc serwer używa bramy `192.168.0.1`.

## Jeżeli nie działa - podstawy adresacji

| Objaw | Hipoteza | Pierwszy test | Prawidłowy wynik | Następny krok |
|---|---|---|---|---|
| Brak łączności z urządzeniem lokalnym | Błędny adres lub maska | `ip -br addr` | Adres i `/24` zgodne z siecią lokalną | Sprawdź link, adresację drugiego urządzenia i VLAN |
| Brak dostępu do Internetu, ale LAN działa | Błędna brama | `ip -4 route` | `default via 192.168.0.1` | Sprawdź osiągalność bramy pingiem |
| Połączenie po IP działa, nazwa nie | Problem DNS | `getent ahostsv4 example.com` | Zwrócone adresy IPv4 | Sprawdź serwer DNS, nie zmieniaj od razu bramy |

## Zadanie rozwiązane

```text
192.168.5.72/24
adres sieci: 192.168.5.0
broadcast:   192.168.5.255
```

## Stan nauki do wznowienia

- Ukończono dzień 3.
- Do powtórki: adres sieci dla `/24` zachowuje pierwsze trzy liczby adresu urządzenia.
- Następny krok: dzień 4 - obliczanie podsieci, VLSM i wstęp do IPv6.
