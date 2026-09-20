# Dzień 4 - Podsieci VLSM i wstęp do IPv6

## Cel

Podział sieci `/24` na mniejsze podsieci, określanie granic `/26`, podstawy VLSM i rozpoznanie najważniejszych cech IPv6.

## Podział `/24` na cztery podsieci `/26`

Podział `192.168.10.0/24` na cztery równe podsieci wymaga zmiany prefiksu na `/26`. Każda podsieć ma 64 adresy, z czego 62 są typowymi adresami urządzeń.

| Podsieć | Zakres | Broadcast | Typowe adresy urządzeń |
|---|---|---|---|
| `192.168.10.0/26` | `.0`–`.63` | `.63` | `.1`–`.62` |
| `192.168.10.64/26` | `.64`–`.127` | `.127` | `.65`–`.126` |
| `192.168.10.128/26` | `.128`–`.191` | `.191` | `.129`–`.190` |
| `192.168.10.192/26` | `.192`–`.255` | `.255` | `.193`–`.254` |

Dla `/26` wielkość bloku wynosi 64. Kolejne adresy sieci to `0`, `64`, `128` i `192` w ostatnim oktecie.

## Ćwiczenie rozwiązane

Adres `192.168.10.100/26` należy do podsieci:

```text
sieć:      192.168.10.64/26
broadcast: 192.168.10.127
hosty:     192.168.10.65–192.168.10.126
```

## VLSM

VLSM (*Variable Length Subnet Mask*) pozwala tworzyć podsieci różnej wielkości. Najpierw przydziela się największe segmenty, a następnie mniejsze.

Przykład dla `10.10.0.0/24`:

| Segment | Potrzebne urządzenia | Najmniejsza podsieć | Przykładowy zakres |
|---|---:|---|---|
| Pracownicy | 50 | `/26` - 62 hosty | `10.10.0.0/26` |
| Goście | 20 | `/27` - 30 hostów | `10.10.0.64/27` |
| Serwery | 10 | `/28` - 14 hostów | `10.10.0.96/28` |

`/26` pomieściłaby 20 urządzeń, ale VLSM wybiera `/27`, bo jest najmniejszą wystarczającą podsiecią.

## Wstęp do IPv6

| Cecha | IPv4 | IPv6 |
|---|---:|---:|
| Długość adresu | 32 bity | 128 bitów |
| Przykład | `192.168.0.51` | `fd01::34b2:a436:f020:d60b/64` |
| Broadcast | Tak | Nie; stosowany jest multicast |
| Typowy prefiks LAN | różny, np. `/24` | zwykle `/64` |

`::` w IPv6 skraca jeden ciąg grup zerowych. Na tym etapie nie zmienialiśmy konfiguracji IPv6.

## Jeżeli nie działa - błąd podsieci

| Objaw | Hipoteza | Pierwszy test | Interpretacja | Następny krok |
|---|---|---|---|---|
| Dwa urządzenia nie komunikują się lokalnie | Różne maski lub błędny adres | Odczytaj IP i prefiks na obu urządzeniach | Adresy mogą wyglądać podobnie, ale należeć do różnych sieci | Porównaj adres sieci dla obu urządzeń |
| Brak dostępu do innej podsieci | Brak lub błędna brama/trasa | `ip -4 route` | Brak trasy domyślnej lub właściwej trasy | Sprawdź bramę i routing po obu stronach |
| Adres ustawiony jako broadcast | Błędne przypisanie IP | Porównaj adres z granicami podsieci | `.127` dla `192.168.10.64/26` to broadcast | Przydziel adres hosta `.65`–`.126` |

## Stan nauki do wznowienia

- Ukończono dzień 4.
- Do powtórki: adres sieci, broadcast i zakres hostów dla `/26`; najmniejszy dobór podsieci w VLSM.
- Następny krok: dzień 5 - MAC, ARP, TCP, UDP, ICMP i porty.
