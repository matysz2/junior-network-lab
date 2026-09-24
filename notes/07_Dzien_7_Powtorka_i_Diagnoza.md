# Dzien 7 - Powtorka i samodzielna diagnoza

## Cel

Utrwalenie podstaw IPv4, podsieci, bramy, DNS, DHCP i SSH oraz dobieranie testu do objawu.

## Wynik quizu

Wynik: **4,5/7 punktow (64%)**.

| Pytanie | Odpowiedz | Ocena | Wniosek |
|---|---|---:|---|
| Adres sieci dla `192.168.10.100/26` | Poczatkowo `.1` | 0/1 | Poprawnie: `192.168.10.64/26` |
| Broadcast dla `192.168.10.64/26` | Poczatkowo `.255` | 0/1 | Poprawnie: `192.168.10.127` |
| Czy `192.168.0.20` jest lokalny dla `192.168.0.51/24`? | Tak | 1/1 | Oba adresy sa w `192.168.0.0/24` |
| Ping do IP dziala, domena nie | DNS | 1/1 | Wlasciwy pierwszy kierunek diagnozy |
| Adres `169.254.10.25` | Brama | 0,5/1 | Najpierw DHCP; brak bramy jest skutkiem |
| Czy ping potwierdza SSH? | Nie | 1/1 | Ping testuje ICMP, SSH TCP 22 |
| Dobor testu przy problemie z domena | `nslookup example.com` | 1/1 | Poprawny test DNS |

## Poprawka - podsieci /26

`/26` oznacza cztery bloki po 64 adresy w sieci `/24`:

| Podsieć | Hosty | Broadcast |
|---|---|---|
| `192.168.10.0/26` | `.1` - `.62` | `.63` |
| `192.168.10.64/26` | `.65` - `.126` | `.127` |
| `192.168.10.128/26` | `.129` - `.190` | `.191` |
| `192.168.10.192/26` | `.193` - `.254` | `.255` |

Dlatego adres `192.168.10.100/26` nalezy do podsieci `192.168.10.64/26`, a jej broadcast to `192.168.10.127`.

## Przeprowadzona diagnoza DNS

Objaw: Internet dziala po adresie IP, ale strona nie otwiera sie po nazwie.

Pierwszy test wskazany samodzielnie:

```bash
nslookup example.com
```

Wykonany test na Ubuntu:

```bash
resolvectl query example.com
```

Wynik: DNS zwrocil adresy IPv4 `104.20.23.154`, `172.66.147.243` oraz IPv6 dla `example.com`; odpowiedz w 21,9 ms.

Wniosek: DNS dziala. Nastepnym testem aplikacji WWW byloby:

```bash
curl -I --max-time 10 https://example.com
```

## Scenariusz z brama

Konfiguracja:

```text
IP:    192.168.5.20/24
Brama: 192.168.0.1
```

Prawidlowa diagnoza: brama jest w innej podsieci niz komputer. Przy masce `/24` brama powinna nalezec do sieci `192.168.5.0/24`, np. `192.168.5.1`.

## Sciagawka diagnostyczna

| Objaw | Pierwszy test | Co oznacza wynik |
|---|---|---|
| Domena nie dziala, IP dziala | `resolvectl query example.com` | Brak odpowiedzi wskazuje na DNS |
| Adres `169.254.x.x` | Sprawdz DHCP i polaczenie | Klient nie otrzymal konfiguracji DHCP |
| Brama w innej podsieci | Porownaj IP, maske i brame | Ruch poza LAN nie wyjdzie poprawnie |
| Ping dziala, SSH nie | `ss -ltn 'sport = :22'` | Sprawdz usluge, TCP 22 i firewall |

## Stan nauki do wznowienia

- Ukonczone dni 1-7.
- Mocne strony: DNS, DHCP/APIPA, rozroznianie ICMP i SSH, podstawowa diagnostyka.
- Do powtorki: wyznaczanie adresu sieci i broadcastu dla `/26`.
- Nastepny dzien: Cisco Packet Tracer, tablica MAC, VLAN, porty access/trunk i 802.1Q.

## GitHub

Zapisz plik jako `labs/07_powtorka_i_diagnoza.md`.

Sugerowany commit:

```text
Dodaj powtorke podstaw i scenariusze diagnostyczne
```
