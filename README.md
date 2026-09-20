# Dzień 1 — inwentaryzacja laboratorium

## Stan środowiska

- Serwer: Ubuntu 24.04.5 LTS.
- Dostęp do serwera: SSH z komputera Windows.
- Procesor serwera: Intel Core i5-3470T, 4 logiczne procesory.
- Pamięć RAM: 7,5 GiB; podczas pomiaru dostępne było około 5,8 GiB.
- Dysk systemowy: około 377 GiB wolnego miejsca.
- Wirtualizacja: Intel VT-x wykryte, urządzenie `/dev/kvm` dostępne.
- Komputer kliencki: Windows 11 Home.
- Sprzęt sieciowy: zwykły router Wi-Fi.

## Wykonane testy

| Test | Wynik |
|---|---|
| Sprawdzenie systemu | Ubuntu 24.04.5 LTS |
| Łączność z bramą | Poprawna |
| Łączność z adresem zewnętrznym | Poprawna |
| Rozwiązywanie nazw DNS | Poprawne dla `example.com` |
| Test HTTPS | `HTTP/2 200` dla `https://example.com` |

## Wnioski

Serwer nadaje się do podstawowych ćwiczeń sieciowych. Z uwagi na 7,5 GiB RAM cięższe maszyny wirtualne będą uruchamiane pojedynczo. Docelowa sieć biura nie została jeszcze zbudowana.

## Następny krok

Dzień 2: podstawowe urządzenia sieciowe, LAN/WAN, Ethernet oraz modele OSI i TCP/IP.