# Postęp nauki sieci — Dzień 26 — 2026-10-10

## Status
Dzień 26 wykonany praktycznie w zakresie NTP, logów/syslog, podstaw SNMP i monitoringu oraz prostego skryptu diagnostycznego Bash.

## Wykonane praktycznie

### NTP
- `timedatectl status`:
  - `System clock synchronized: yes`
  - `NTP service: active`
  - strefa: `Europe/Warsaw (CEST, +0200)`
- `systemd-timesyncd` działa jako aktywna usługa.
- Aktualny serwer czasu: `ntp.ubuntu.com` / `185.125.190.58:123`.
- `timedatectl timesync-status`:
  - Stratum: 2
  - Offset: +4.063 ms
  - Delay: 38.078 ms
  - Jitter: 1.700 ms

### Logi / syslog
- `rsyslog` działa: `active (running)`.
- Utworzono testowy wpis:
  - `logger -t dzien26 "Test syslog - Dzien 26"`
- Potwierdzono wpis w:
  - `journalctl -t dzien26 --no-pager`
  - `/var/log/syslog`

### SNMP
- Początkowo `snmpd` nie był zainstalowany.
- Zainstalowano pakiety `snmpd` i `snmp`.
- `snmpd` działa: `active (running)`.
- Agent domyślnie nasłuchuje tylko lokalnie:
  - `127.0.0.1:161`
  - `[::1]:161`
- Potwierdzono lokalny `snmpwalk` dla gałęzi `system`.
- Zweryfikowano ograniczenie domyślnego widoku `systemonly`.
- Dodano osobny plik laboratoryjny:
  - `/etc/snmp/snmpd.conf.d/day26-lab.conf`
- Konfiguracja lab:
  - widok `day26view`
  - system + interfaces
  - `rocommunity dzien26ro 127.0.0.1 -V day26view`
- Po restarcie `snmpd` działa poprawnie.
- Odczytano listę interfejsów przez `ifDescr`.
- Odczytano `ifOperStatus`:
  - `eno1 = 1 (up)`
  - `virbr20 = 2 (down)`
- Odczytano liczniki `ifInOctets` i `ifOutOctets` dla `eno1`.
- Po wygenerowaniu ruchu liczniki wzrosły, co potwierdziło praktyczny mechanizm monitoringu ruchu.

### Skrypt diagnostyczny Bash
Utworzono:
- `~/dzien26-diagnostyka.sh`

Skrypt sprawdza:
- adresy IPv4,
- bramę domyślną,
- ping do bramy,
- Internet po IP,
- DNS,
- synchronizację NTP,
- usługę rsyslog,
- usługę snmpd,
- lokalne zapytanie SNMP.

Wszystkie testy zakończyły się `OK`.

## Dodatkowe zagadnienia
- REST API: programistyczny sposób pobierania/zmiany danych przez żądania HTTP.
- JSON: popularny format danych używany przez API.
- Ansible: automatyzacja konfiguracji wielu urządzeń/systemów w sposób deklaratywny.
- Terraform: Infrastructure as Code, głównie do tworzenia i utrzymywania infrastruktury.
- AI w administracji siecią: pomoc przy analizie logów, tworzeniu skryptów i dokumentacji; wynik AI wymaga weryfikacji przed użyciem produkcyjnym.

## Quiz
Wynik: 5/6 punktów (~83%).

### Do powtórki
- `ifOperStatus = 1` oznacza `up`.
- Oddzielaj test łączności IP od testu DNS:
  - IP działa + DNS nie działa => problem DNS.
  - IP nie działa => problem jest wcześniej, np. adresacja, brama, routing, firewall lub łącze.
- Nie nazywaj testu `ping 1.1.1.1` testem warstwy transportowej — ICMP działa w warstwie sieciowej.

## Konfiguracja pozostawiona po laboratorium
- `snmpd` zainstalowany i aktywny.
- Dostęp SNMP pozostaje lokalny (`127.0.0.1` / `::1`).
- Dodatkowa konfiguracja lab: `/etc/snmp/snmpd.conf.d/day26-lab.conf`.
- Community `dzien26ro` jest używane tylko lokalnie.
- Skrypt `~/dzien26-diagnostyka.sh` pozostaje jako materiał do portfolio.

## Rollback SNMP lab
Aby usunąć dodatkową konfigurację laboratoryjną:
```bash
sudo rm /etc/snmp/snmpd.conf.d/day26-lab.conf
sudo systemctl restart snmpd
```

## Następny krok
Dzień 27: scalenie projektu małej firmy — segmenty użytkownicy/serwery/goście, DNS/DHCP, routing, firewall, VPN, plan adresacji i dokumentacja; tylko elementy wykonalne na sprzęcie, pozostałe oznaczyć jako symulację.
