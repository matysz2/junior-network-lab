# Stan nauki do wznowienia - 25.09.2026

## Etap kursu
Dzień 10, dział 2, ukończony praktycznie. Wykonano pętle i redundancję, STP/RSTP, EtherChannel z LACP, CDP/LLDP oraz port-security. Wynik pytań podstawowych był poniżej 80%, dlatego dzień 11 należy zacząć od krótkiej powtórki ról STP, różnicy STP/EtherChannel i skutków naruszenia port-security.

## Aktualna topologia
- Switch0 i Switch1: Cisco 2960.
- Router1: Cisco 2911, router-on-a-stick przez Switch0 G0/1 - Router G0/0.
- VLAN 10 `PRACOWNICY`, VLAN 20 `GOSCIE`.
- PC0: Switch0 Fa0/1, VLAN 10, 192.168.10.10/24, brama 192.168.10.1.
- PC1: Switch0 Fa0/2, VLAN 20, 192.168.20.20/24, brama 192.168.20.1.
- PC2: Switch1 Fa0/1, VLAN 10, 192.168.10.30/24.
- Router G0/0.10 = 192.168.10.1/24, G0/0.20 = 192.168.20.1/24.

## STP i RSTP - wykonane
- Dwa osobne trunki Fa0/23 i Fa0/24 początkowo tworzyły redundantne połączenia między switchami.
- Switch0 był root bridge dla VLAN 1, 10 i 20.
- Przy klasycznym STP na Switch1: Fa0/23 `Root FWD`, Fa0/24 `Altn BLK`.
- Po awarii aktywnego łącza klasyczny STP przeszedł przez `Listening`, a następnie uruchomił Fa0/24 jako `Root FWD`.
- Oba switche przełączono na `spanning-tree mode rapid-pvst`.
- Przy RSTP kontrolowane `shutdown` Fa0/23 szybko przełączyło ruch na Fa0/24.
- Ping PC2 -> PC0 po przełączeniu: 4/4 odpowiedzi, 0% strat.

## EtherChannel i LACP - stan końcowy
- Fa0/23 i Fa0/24 na obu switchach połączono w `channel-group 1 mode active`.
- LACP działa poprawnie po obu stronach.
- `show etherchannel summary` potwierdziło:
  - `Po1(SU)` - kanał warstwy 2 jest używany,
  - `Fa0/23(P)` i `Fa0/24(P)` - oba porty należą do kanału.
- `show interfaces trunk` na Switch0:
  - Po1 trunking, 802.1Q, native VLAN 1,
  - VLAN-y allowed/active/forwarding: 10,20,
  - G0/1 do routera pozostaje osobnym trunkiem VLAN 10,20.
- STP widzi EtherChannel jako jeden port logiczny:
  - na Switch0 Po1 `Desg FWD`, koszt 12,
  - na Switch1 Po1 `Root FWD`, koszt 12.
- Test awarii jednego członka: na Switch1 wyłączono Fa0/23; wynik `Po1(SU)`, Fa0/23(D), Fa0/24(P).
- Podczas awarii ping PC2 -> PC0: 4/4 odpowiedzi, 0% strat.
- Fa0/23 przywrócono przez `no shutdown`; końcowo oba porty mają `(P)`.

## CDP i LLDP
- `show cdp neighbors` wykrył sąsiedni switch Cisco 2960. Po przebudowie łączy widoczne były także stare wpisy do wygaśnięcia Holdtime.
- LLDP włączono globalnie przez `lldp run` na obu switchach.
- `show lldp neighbors` na Switch1 pokazał sąsiada przez Po1 i fizyczne porty Fa0/23 oraz Fa0/24.
- CDP jest rozwiązaniem Cisco; LLDP jest standardem otwartym dla urządzeń różnych producentów.

## Port-security - stan końcowy
- Na Switch1 Fa0/1 w VLAN 10 skonfigurowano:
  - `switchport port-security`,
  - `maximum 1`,
  - `mac-address sticky`,
  - `violation shutdown`.
- Po wygenerowaniu ruchu z PC2 switch nauczył się sticky MAC `0005.5E80.C834` w VLAN 10.
- Stan: Port Security Enabled, Secure-up, Sticky MAC Addresses 1, Security Violation Count 0.
- Przy innym MAC port zostałby wyłączony ochronnie i zablokowałby cały ruch, nie tylko Internet.
- Po usunięciu przyczyny port można przywrócić sekwencją `shutdown` i `no shutdown`.

## Błędy i wnioski
- Po konfiguracji LACP tylko na Switch0 porty były chwilowo suspended/stand-alone, ponieważ zdalne porty nie miały jeszcze LACP. Po konfiguracji Switch1 kanał przeszedł do `Po1(SU)` i porty do `(P)`.
- Użytkownik początkowo cofnął `channel-group`, interpretując oczekiwany komunikat o braku LACP po drugiej stronie jako awarię. Konfigurację wykonano ponownie poprawnie.
- Wynik STP ze Switch0 został początkowo potraktowany jak wynik Switch1. Rozpoznanie urządzenia: Switch0 miał dodatkowo Gi0/1 do routera i Po1 `Desg FWD`; Switch1 miał Po1 `Root FWD`.

## Pytania i ocena
- Pytania 1-6 z części STP/RSTP: 4,5/6.
- Różnica STP i EtherChannel: brak odpowiedzi - 0/1.
- Skutek `maximum 1` + `violation shutdown`: odpowiedź częściowa, że nowe urządzenie nie będzie miało Internetu; poprawka: switch wyłącza cały port - 0,5/1.
- Wynik podstawowych pytań dnia 10: 5/8 = 62,5%.
- Poprawka po powtórce: użytkownik prawidłowo wskazał, że STP widzi jeden logiczny port Po1.
- Ocena: praktyczne ćwiczenia wykonane poprawnie z prowadzeniem; teoria wymaga krótkich powtórek.

## Zapis konfiguracji
- Switch0 i Switch1 zapisano poleceniem `copy running-config startup-config`; użytkownik potwierdził `[OK]`/zapis.

## Następny krok
Rozpocząć dzień 11 krótką powtórką bez ponownego wykonywania dnia 10:
1. `Root FWD` kontra `Altn BLK`.
2. Dwa osobne trunki ze STP kontra jeden Po1 z LACP.
3. Skutek `violation shutdown`.
Następnie dzień 11: WAN, podstawy OSPF w jednym obszarze, wstęp do tras IPv6 i redundancji bramy. Pytania i czynności podawać pojedynczo.
