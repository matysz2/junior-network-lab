# Stan nauki do wznowienia — 24.09.2026

## Etap kursu
Dzień 9, dział 2: routing między VLAN-ami metodą router-on-a-stick w Cisco Packet Tracer. Laboratorium praktyczne zakończone powodzeniem. Dział 2 (dni 8–12) nadal trwa.

## Topologia i potwierdzona konfiguracja
- Switch0 i Switch1: Cisco 2960. Łącze Switch0 Fa0/24–Switch1 Fa0/24 działa jako trunk 802.1Q; dozwolone VLAN-y 10 i 20, native VLAN 1.
- Router1: Cisco 2911. Router G0/0 jest połączony ze Switch0 G0/1. Port Switch0 G0/1 skonfigurowano jako trunk z VLAN 10 i 20.
- VLAN 10 `PRACOWNICY`; VLAN 20 `GOSCIE`.
- PC0: Switch0 Fa0/1, VLAN 10, 192.168.10.10/24, brama 192.168.10.1.
- PC1: Switch0 Fa0/2, VLAN 20, 192.168.20.20/24, brama 192.168.20.1.
- PC2: Switch1 Fa0/1; wcześniej potwierdzony w VLAN 10 z adresem 192.168.10.30/24. Brama PC2 nie była dziś sprawdzana.

## Router-on-a-stick
- Interfejs fizyczny G0/0: `no shutdown`, bez adresu IP, stan up/up.
- G0/0.10: `encapsulation dot1Q 10`, adres 192.168.10.1/24, stan up/up.
- G0/0.20: `encapsulation dot1Q 20`, adres 192.168.20.1/24, stan up/up.
- `show ip route` potwierdziło sieci bezpośrednio podłączone 192.168.10.0/24 przez G0/0.10 i 192.168.20.0/24 przez G0/0.20 oraz adresy lokalne /32.
- Brak gateway of last resort jest prawidłowy dla tego laboratorium; statyczna trasa nie jest potrzebna między dwiema bezpośrednio podłączonymi sieciami.

## Testy i wyniki
- PC1 → 192.168.20.1: 4/4 odpowiedzi, 0% strat.
- PC1 (VLAN 20) → PC0 192.168.10.10 (VLAN 10): 4/4 odpowiedzi, 0% strat. Routing między VLAN-ami działa.
- Konfigurację routera i Switch0 zapisano poleceniem `copy running-config startup-config`; użytkownik potwierdził zapis. Switch1 był zapisany wcześniej.

## Błędy szkoleniowe i wyjaśnienia
- Polecenia podinterfejsu routera wpisano początkowo na switchu. IOS je odrzucił; niczego nie uszkodzono. Utrwalić rozpoznawanie promptów `Switch>` / `Router>`.
- `copy running-config startup-config` wpisano przy `Router>`; wymagane jest najpierw `enable`, aby uzyskać `Router#`.
- `end` wpisane w trybie `Switch>` zostało potraktowane jako nazwa hosta. `end` służy do wyjścia z trybu konfiguracji.
- `clear` nie czyści ekranu Cisco CLI bez dodatkowego argumentu, a `clean` nie jest poleceniem IOS. Wyszukiwanie DNS można przerwać Ctrl+Shift+6.

## Pytania kontrolne
- Poprzednie pytanie dnia 9: czy VLAN 10 i VLAN 20 komunikują się bez routera — poprawnie: nie (1/1).
- Dlaczego PC1 używa bramy dla PC0 w innej sieci — odpowiedź częściowa (0,5/1); utrwalić rozpoznawanie innej podsieci na podstawie maski.
- Brama PC1 w VLAN 20 — poprawnie 192.168.20.1 (1/1).
- Znaczenie `encapsulation dot1Q 20` — brak odpowiedzi (0/1); utrwalić, że podinterfejs obsługuje ramki oznaczone VLAN 20.
- Podinterfejs VLAN 10 — poprawnie G0/0.10 (1/1).
- Port switcha do routera dla dwóch VLAN-ów — poprawnie trunk (1/1).
- Polecenie pokazujące sieci znane routerowi — błędnie `ping`; poprawnie `show ip route` (0/1).
- Wynik łączny: 4,5/7. Praktyka wykonana poprawnie, teoria wymaga krótkiej powtórki.

## Następny krok
Na początku kolejnej sesji krótko powtórzyć: `ping` kontra `show ip route`, działanie `encapsulation dot1Q`, rolę maski i bramy. Następnie sprawdzić konfigurację poleceniami `show interfaces trunk`, `show ip interface brief`, `show ip route` i przejść do kolejnego punktu dnia 9/10 programu bez rozpoczynania kursu od początku. Pytania zadawać pojedynczo.
