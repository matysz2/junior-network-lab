# Dzien 1 - Inwentaryzacja laboratorium

## Cel

Ustalenie rzeczywistego stanu laboratorium i sprawdzenie podstawowej lacznosci sieciowej z serwera Ubuntu.

## Stan potwierdzony podczas sesji

| Element | Wynik |
|---|---|
| Serwer | Ubuntu 24.04.5 LTS |
| Dostep | SSH z komputera z Windows 11 Home |
| Procesor | Intel Core i5-3470T, 4 logiczne procesory |
| Pamięc RAM | 7,5 GiB lacznie; okolo 5,8 GiB `available` w chwili pomiaru |
| Dysk systemowy | `/dev/sdb2`, 377 GiB wolnego miejsca |
| Interfejs LAN | `eno1`, adres `192.168.0.51/24` |
| Brama domyslna | `192.168.0.1` |
| DNS dla `eno1` | `192.168.0.1` |
| Wirtualizacja | Intel VT-x; urzadzenie `/dev/kvm` jest dostepne |
| Siec | zwykly router Wi-Fi; brak switcha zarzadzanego |

## Wykonane testy

| Test | Polecenie | Wynik rzeczywisty | Wniosek |
|---|---|---|---|
| Informacja o systemie | `cat /etc/os-release` | Ubuntu 24.04.5 LTS | Serwer nie jest Debianem, ale jest systemem z rodziny Debian. |
| Pamiec | `free -h` | 5,8 GiB w kolumnie `available` | Zasoby wystarcza na lekkie laboratorium; ciezsze VM beda uruchamiane pojedynczo. |
| Dysk | `df -h /` | 377 GiB dostepne | Jest miejsce na obrazy VM i materialy laboratoryjne. |
| Konfiguracja IP | `ip -br addr` | `eno1` UP, `192.168.0.51/24` | Interfejs Ethernet jest aktywny. |
| Trasa domyslna | `ip -4 route` | przez `192.168.0.1` | Brama kieruje ruch do innych sieci. |
| Lacznosc lokalna | `ping -c 4 192.168.0.1` | 4/4 odpowiedzi, 0% strat | Brama odpowiada. |
| Lacznosc zewnetrzna | `ping -c 4 1.1.1.1` | 4/4 odpowiedzi, 0% strat | Osiagalny jest sprawdzony adres poza LAN. |
| DNS | `getent ahostsv4 example.com` | Zwrocone dwa adresy IPv4 | System rozwiazuje nazwe domenowa. |
| HTTPS | `curl -I --max-time 10 https://example.com` | `HTTP/2 200` | Usluga HTTPS odpowiedziala poprawnie. |

## Najwazniejsze pojecia

- **DNS** zamienia nazwe, np. `example.com`, na adres IP.
- **Maska/prefiks**, np. `/24`, okresla czesc sieciowa adresu IP.
- **Brama domyslna** przekazuje ruch do innych sieci, np. w kierunku Internetu.
- `free` oznacza pamiec calkowicie wolna, a `available` szacowana pamiec dostepna dla programow bez uzycia swapu.

## Ograniczenia i decyzje laboratoryjne

- Windows 11 Home nie zawiera pelnego Hyper-V.
- Windows Server uruchomimy pozniej jako jedna maszyna wirtualna na Ubuntu przez KVM/QEMU.
- Z powodu 7,5 GiB RAM nie bedziemy jednoczesnie uruchamiac kilku ciezkich maszyn.
- VLAN-y i switching Cisco przeprowadzimy w Cisco Packet Tracerze; fizyczny switch zarzadzany nie jest teraz wymagany.

## Powtorka

Do ponownego sprawdzenia bez podpowiedzi: roznica miedzy DNS, maska i brama oraz miedzy `free` i `available`.

## Nastepny krok

Dzien 2: LAN, WAN, router, switch, access point, Ethernet oraz modele OSI i TCP/IP.
