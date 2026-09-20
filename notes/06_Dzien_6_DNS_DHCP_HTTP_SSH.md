# Dzien 6 - DNS, DHCP, HTTP/HTTPS i SSH

## Cel

Rozroznianie problemow z DNS, DHCP, strona WWW i usluga SSH oraz wykonanie bezpiecznych testow diagnostycznych.

## Wykonane testy

### 1. DNS

Polecenie:

```bash
resolvectl query example.com
```

Wynik: nazwa `example.com` zostala rozwiazana przez DNS przez interfejs `eno1`. Otrzymano adresy IPv4 `104.20.23.154`, `172.66.147.243` oraz adresy IPv6. Czas odpowiedzi: 19,3 ms.

Wniosek: DNS w laboratorium dziala.

### 2. DHCP i trasa domyslna

Polecenie:

```bash
ip route show default
```

Wynik:

```text
default via 192.168.0.1 dev eno1 proto dhcp src 192.168.0.51 metric 100
```

Wniosek: brama to `192.168.0.1`, adres zrodlowy hosta to `192.168.0.51`, a wpis pochodzi z DHCP (`proto dhcp`).

## DHCP - DORA

1. **Discover** - klient szuka serwera DHCP.
2. **Offer** - serwer proponuje adres i ustawienia.
3. **Request** - klient prosi o proponowana konfiguracje.
4. **ACK** - serwer zatwierdza konfiguracje.

Adres `169.254.x.x` (APIPA) nie pochodzi z routera. Oznacza, ze klient nie otrzymal odpowiedzi DHCP.

## HTTP i HTTPS

Polecenie:

```bash
curl -I --max-time 10 https://example.com
```

Wynik: `HTTP/2 200`.

Wniosek: serwer odpowiedzial poprawnie przez HTTPS. `200` oznacza sukces, a HTTPS standardowo korzysta z TCP 443.

## SSH

Polecenie:

```bash
ss -ltn 'sport = :22'
```

Wynik: `LISTEN` dla `0.0.0.0:22` i `[::]:22`.

Wniosek: SSH nasluchuje na TCP 22 na wszystkich adresach IPv4 i IPv6 hosta.

`ping` testuje ICMP, dlatego sam nie potwierdza dzialania SSH ani portu 22.

## Szybka diagnostyka

| Objaw | Pierwszy podejrzany obszar | Test |
|---|---|---|
| Dziala ping do 1.1.1.1, nie dziala domena | DNS | `resolvectl query example.com` |
| Brak IPv4 lub adres 169.254.x.x | DHCP lub polaczenie z siecia | Windows: `ipconfig /all` |
| Strona nie odpowiada | HTTP/HTTPS | `curl -I --max-time 10 https://example.com` |
| Nie dziala SSH | Usluga, TCP 22, firewall | `ss -ltn 'sport = :22'` |

## Najwazniejsze wnioski

- DNS zamienia nazwy domenowe na adresy IP.
- DHCP automatycznie przekazuje adres IP, maske, brame i zwykle DNS.
- DORA konczy sie komunikatem ACK.
- `HTTP/2 200` oznacza prawidlowa odpowiedz serwera WWW.
- SSH wymaga dzialajacej uslugi i dostepnego portu TCP 22; ping nie wystarcza.

## GitHub

Zapisz plik w repozytorium jako `labs/06_dns_dhcp_http_ssh.md`.

Sugerowany commit:

```text
Dodaj testy DNS DHCP HTTP i SSH
```
