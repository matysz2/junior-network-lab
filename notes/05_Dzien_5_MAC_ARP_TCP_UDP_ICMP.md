# Dzień 5 - MAC ARP TCP UDP ICMP i porty

## Cel

Rozróżnianie adresu MAC i IP, rozumienie ARP, TCP, UDP, ICMP oraz weryfikacja usługi po właściwym porcie.

## MAC i ARP

| Pojęcie | Warstwa | Rola |
|---|---:|---|
| Adres IP | 3 | Określa urządzenie i umożliwia routing między sieciami |
| Adres MAC | 2 | Dostarcza ramkę Ethernet w lokalnym segmencie |
| ARP | 2/3 | Ustala MAC dla znanego adresu IPv4 w lokalnym LAN |

Gdy Ubuntu wysyła ruch do `1.1.1.1`, szuka przez ARP adresu MAC bramy `192.168.0.1`, a nie adresu MAC serwera `1.1.1.1`.

Wykonano:

```bash
ip neigh show dev eno1
```

Wynik zawierał bramę `192.168.0.1` ze stanem `REACHABLE`, co potwierdza aktualny wpis sąsiada z adresem MAC w tablicy ARP.

## TCP UDP ICMP i porty

| Protokół | Znaczenie | Przykłady |
|---|---|---|
| TCP | Połączeniowy, potwierdza dostarczenie | SSH 22, HTTPS 443, RDP 3389 |
| UDP | Lżejszy, bez wbudowanego potwierdzania | DNS 53, część transmisji czasu rzeczywistego |
| ICMP | Komunikaty diagnostyczne i kontrolne | ping, `echo request`, `echo reply` |

Ping nie potwierdza działania SSH, ponieważ ping używa ICMP, a SSH TCP na porcie 22.

Wykonano:

```bash
ss -ltn 'sport = :22'
```

Wynik `LISTEN 0.0.0.0:22` oraz `LISTEN [::]:22` potwierdził, że SSH nasłuchuje na wszystkich adresach IPv4 i IPv6 serwera.

## Przechwyt pakietów ICMP

W drugim oknie SSH wykonano:

```bash
sudo tcpdump -ni eno1 -c 4 icmp
```

Równocześnie wygenerowano dwa pingi do bramy:

```bash
ping -c 2 192.168.0.1
```

Zaobserwowano dwa żądania i dwie odpowiedzi:

```text
192.168.0.51 > 192.168.0.1: ICMP echo request
192.168.0.1 > 192.168.0.51: ICMP echo reply
```

`echo request` oznacza żądanie Ubuntu. `echo reply` oznacza odpowiedź bramy.

## Jeżeli nie działa - usługa SSH

| Objaw | Hipoteza | Pierwszy test | Prawidłowy wynik | Następny krok |
|---|---|---|---|---|
| Ping działa, SSH nie | SSH nie nasłuchuje lub port jest blokowany | `ss -ltn 'sport = :22'` | `LISTEN` na porcie 22 | Sprawdź usługę SSH i zaporę dla TCP 22 |
| Nie ma wpisu ARP bramy | Brak ruchu albo problem L2 | `ip neigh show dev eno1` | Wpis bramy z MAC | Wygeneruj ping do bramy; sprawdź kabel i VLAN, gdy wpis nie powstaje |
| Ping nie odpowiada | ICMP blokowany lub problem łączności | Sprawdź stan interfejsu i trasę | `UP,LOWER_UP`, poprawna brama | Dobierz następny test do usługi, nie zakładaj awarii urządzenia |

## Stan nauki do wznowienia

- Ukończono dzień 5.
- Do powtórki: ping testuje ICMP, a nie konkretną usługę TCP; `echo reply` to odpowiedź.
- Następny krok: dzień 6 - DNS, DHCP, DORA, HTTP/HTTPS i SSH.
