# Stan nauki do wznowienia — 21.09.2026

## Etap kursu
Dzień 8, dział 2: Cisco CLI, VLAN, access/trunk, 802.1Q i native VLAN. Sesja zakończona na prośbę użytkownika. Nie uznajemy całego dnia ani działu za zaliczony. Program kursu Prompt_Nauka_Sieci_28_Dni.txt został odczytany w tej rozmowie.

## Laboratorium i potwierdzone wyniki
- Packet Tracer: Switch0 i Switch1 (2960), połączenie Fa0/24–Fa0/24; PC0 i PC1 przy Switch0, PC2 przy Switch1 na Fa0/1.
- Plan adresacji używany w ćwiczeniu: PC0 192.168.10.10/24, PC1 192.168.10.20/24, PC2 192.168.10.30/24. W tej części rozmowy nie otrzymano osobnego wyniku ipconfig z PC2; adres PC2 był zadany instrukcją.
- VLAN 10 PRACOWNICY i VLAN 20 GOSCIE utworzone. Trunk dopuszcza 10,20; native VLAN 1.
- Ostatni wynik show interfaces trunk z Switch1: Fa0/24 on, 802.1q, trunking, native 1; VLAN-y allowed, active i forwarding: 10,20.
- PC2 w VLAN 10: ping do PC0 zakończony 4/4 odpowiedzi, 0% strat.
- Po przeniesieniu Fa0/1 Switch1 do VLAN 20: show vlan brief potwierdziło zmianę; ping PC2 do PC0 0/4 odpowiedzi, 100% strat.
- Po instrukcji przywrócenia Fa0/1 do VLAN 10: najpierw 1/4 odpowiedzi, następnie 4/4, 0% strat. Przyczyny przejściowych strat nie ustalono. Ostatniej kontroli show vlan brief po cofnięciu zmiany jeszcze nie wykonano lub nie przesłano.
- Użytkownik potwierdził wcześniejszy zapis konfiguracji obu switchy. Na Switch1 pokazano [OK]; na drugim konieczne było enable przed copy running-config startup-config. Zapis pliku .pkt zalecano; brak jednoznacznego osobnego potwierdzenia tego zapisu.

## Umiejętności i trudności
- Po powtórkach poprawnie rozróżnia access dla zwykłego PC i trunk dla wielu VLAN-ów.
- Po wyjaśnieniach poprawnie wskazuje, że trunk zachowuje rozdzielenie VLAN-ów; do komunikacji między nimi potrzebny routing i właściwa adresacja.
- Znacznik 802.1Q oraz odczytywanie native VLAN wymagały wielokrotnych przykładów. Potem poprawne odpowiedzi: tag 10 → VLAN 10, tag 20 → VLAN 20, port access do zwykłego PC bez znacznika, native VLAN 1, allowed VLAN 10 i 20.
- Poprawnie wybrał show vlan brief do sprawdzenia przypisania portu. show interfaces trunk początkowo nie pamiętał.
- Próba samodzielnej zmiany VLAN-u nie powiodła się: użył zapisu konfiguracji zamiast zmiany. Wykonał zmianę poprawnie po podaniu poleceń. Konfiguracja nadal wymaga wsparcia; nie deklarować samodzielnego opanowania.
- Błąd zapisu w trybie Switch> rozwiązany instrukcją enable → Switch#.

## Następny krok
Krótka powtórka, bez rozpoczynania kursu od początku. Na Switch1 wykonać show vlan brief i poprosić użytkownika o wskazanie VLAN-u portu Fa0/1; oczekiwany VLAN 10, zweryfikować wynik. Następnie utrwalić różnicę show vlan brief / show interfaces trunk i zapis switcha / zapis .pkt. Ocenić gotowość do zakończenia dnia 8 przed routingiem z dnia 9. Pytania pojedynczo, bez długiej serii powtarzających się quizów.

## Materiały i portfolio
W tej sesji nie utworzono PDF-u działu, pliku .pkt po stronie asystenta ani zmian w repozytorium GitHub. Dział 2 obejmuje dni 8–12 i nie jest ukończony. Niniejsza notatka dokumentuje rzeczywiste wyniki; nie jest dowodem edycji urządzeń przez asystenta.
