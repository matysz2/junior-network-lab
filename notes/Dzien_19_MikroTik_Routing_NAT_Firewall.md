# Dzień 19 - MikroTik: routing, NAT i firewall

> Dział 5 kursu: MikroTik. Materiał opisuje tylko czynności faktycznie wykonane w laboratorium. RouterOS 7.23.7 long-term.

## Cel dnia

Dzisiaj nie chodziło tylko o wpisanie kilku komend. Celem było zrozumienie **po co router potrzebuje routingu, NAT-u i firewalla oraz jak odróżnić awarię jednego mechanizmu od drugiego**.

## Topologia

```text
Internet / 1.1.1.1
        |
        v
sieć KVM 192.168.122.0/24
        |
gateway 192.168.122.1
        |
ether1 CHR = 192.168.122.75
        |
        +-- VLAN10 PRACOWNICY 192.168.10.0/24
        |      klient10 = 192.168.10.200
        |      gateway  = 192.168.10.1
        |
        +-- VLAN20 GOSCIE 192.168.20.0/24
               klient20 = 192.168.20.200
               gateway  = 192.168.20.1

Dodatkowo:
VLAN30 KAMERY    = 192.168.30.0/24
VLAN40 TELEFONY  = 192.168.40.0/24
```

## Najważniejszy skrót

```text
routing  = wybiera drogę
NAT      = zmienia adres
firewall = pozwala albo blokuje

INPUT   = do routera
FORWARD = przez router
OUTPUT  = z routera
```

## Po co to robimy?

### Po co sprawdzamy tablicę routingu?
Żeby wiedzieć, czy router ma drogę do sieci lokalnych i do Internetu. Bez poprawnej trasy NAT ani firewall nie naprawią braku drogi.

### Po co NAT?
Prywatne adresy klientów nie są routowane w publicznym Internecie. W naszym labie dodatkowo upstream 192.168.122.1 nie ma trasy zwrotnej do 192.168.10.0/24 i 192.168.20.0/24. Masquerade sprawia, że ruch wychodzi z adresem ether1 i odpowiedź wraca do CHR.

### Po co firewall forward?
Żeby kontrolować ruch pomiędzy sieciami i hostami przechodzący przez router, np. goście -> pracownicy.

### Po co firewall input?
Żeby chronić sam router i jego usługi zarządcze, np. WinBox/SSH.

### Po co established,related?
Żeby stateful firewall przepuszczał odpowiedzi należące do już rozpoczętych lub powiązanych połączeń, zamiast analizować je jak nowe połączenia.

### Po co liczniki reguł?
Żeby mieć dowód, że konkretna reguła faktycznie łapie pakiety. To szybka diagnostyka kolejności i dopasowania reguł.

### Po co celowo wyłączaliśmy NAT?
Żeby przećwiczyć diagnostykę realnej awarii w kontrolowanych warunkach i nauczyć się odróżniać routing od NAT.

## Tablica routingu

| Trasa | Typ | Znaczenie |
|---|---|---|
| 0.0.0.0/0 -> 192.168.122.1 | dynamiczna DHCP / default | Jeśli nie ma bardziej szczegółowej trasy, wyślij pakiet do 192.168.122.1. |
| 192.168.10.0/24 -> vlan10-pracownicy | connected | VLAN10 jest bezpośrednio podłączony do routera. |
| 192.168.20.0/24 -> vlan20-goscie | connected | VLAN20 jest bezpośrednio podłączony do routera. |
| 192.168.30.0/24 -> vlan30-kamery | connected | VLAN30 jest bezpośrednio podłączony. |
| 192.168.40.0/24 -> vlan40-telefony | connected | VLAN40 jest bezpośrednio podłączony. |
| 192.168.122.0/24 -> ether1 | connected | Sieć WAN/lab KVM jest bezpośrednio podłączona. |

## Definicje

- **Routing** - Wybór drogi, którą router ma wysłać pakiet do sieci docelowej.
- **Default route 0.0.0.0/0** - Trasa używana, gdy router nie ma bardziej szczegółowego wpisu dla celu.
- **Connected route** - Trasa dodana automatycznie, gdy router ma aktywny interfejs z adresem w danej podsieci.
- **NAT** - Mechanizm zmiany adresów IP w pakietach. W naszym labie używamy source NAT dla ruchu wychodzącego.
- **srcnat** - Łańcuch NAT dotyczący zmiany adresu źródłowego pakietu.
- **Masquerade** - Odmiana srcnat, która używa adresu IP interfejsu wyjściowego. Dobrze pasuje do interfejsu z dynamicznym adresem.
- **Connection tracking** - Mechanizm śledzący połączenia i ich stan. NAT i stateful firewall korzystają z tych informacji.
- **input** - Ruch skierowany DO samego routera.
- **forward** - Ruch PRZECHODZĄCY PRZEZ router do innego hosta/sieci.
- **output** - Ruch wygenerowany PRZEZ sam router i wychodzący z niego.
- **established** - Pakiet należy do już istniejącego połączenia.
- **related** - Pakiet należy do połączenia powiązanego z już istniejącym.
- **drop** - Firewall odrzuca pakiet bez dalszego przetwarzania w danym łańcuchu.

## Komendy i dokładne wyjaśnienia

### 1. `/ip route print`
- **Gdzie:** RouterOS
- **Co robi:** Pokazuje tablicę routingu.
- **Co zobaczyliśmy:** Sprawdziliśmy default route i connected routes.
- **Cofnięcie / uwaga:** `Tylko odczyt.`

### 2. `/ping 1.1.1.1 count=4`
- **Gdzie:** RouterOS
- **Co robi:** Testuje dostęp samego MikroTika do Internetu po IP.
- **Co zobaczyliśmy:** 4/4 odpowiedzi, 0% strat.
- **Cofnięcie / uwaga:** `Tylko test.`

### 3. `sudo ip netns exec klient10 ping -c 4 1.1.1.1`
- **Gdzie:** Ubuntu
- **Co robi:** Testuje Internet z klienta VLAN10.
- **Co zobaczyliśmy:** Przed NAT: 100% strat. Po NAT: 0% strat.
- **Cofnięcie / uwaga:** `Tylko test.`

### 4. `/ip firewall nat print`
- **Gdzie:** RouterOS
- **Co robi:** Pokazuje reguły NAT.
- **Co zobaczyliśmy:** Na początku brak reguł.
- **Cofnięcie / uwaga:** `Tylko odczyt.`

### 5. `/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="NAT VLANs to Internet"`
- **Gdzie:** RouterOS
- **Co robi:** Dodaje source NAT dla ruchu wychodzącego przez ether1.
- **Co zobaczyliśmy:** Po dodaniu klient10 i klient20 uzyskali dostęp do 1.1.1.1.
- **Cofnięcie / uwaga:** `/ip firewall nat remove [find comment="NAT VLANs to Internet"]`

### 6. `sudo ip netns exec klient20 ping -c 4 1.1.1.1`
- **Gdzie:** Ubuntu
- **Co robi:** Test Internetu z VLAN20.
- **Co zobaczyliśmy:** 4/4, 0% strat.
- **Cofnięcie / uwaga:** `Tylko test.`

### 7. `sudo ip netns exec klient20 ping -c 4 192.168.10.200`
- **Gdzie:** Ubuntu
- **Co robi:** Test izolacji gości od pracowników.
- **Co zobaczyliśmy:** 0/4, 100% strat - blokada działa.
- **Cofnięcie / uwaga:** `Tylko test.`

### 8. `sudo ip netns exec klient10 ping -c 4 192.168.20.200`
- **Gdzie:** Ubuntu
- **Co robi:** Test kierunku VLAN10 -> VLAN20.
- **Co zobaczyliśmy:** 4/4, 0% strat.
- **Cofnięcie / uwaga:** `Tylko test.`

### 9. `/ip firewall filter print stats`
- **Gdzie:** RouterOS
- **Co robi:** Pokazuje liczniki reguł filtra.
- **Co zobaczyliśmy:** ALLOW established/related: 28 pakietów; BLOCK VLAN20->VLAN10: 16 pakietów.
- **Cofnięcie / uwaga:** `Tylko odczyt.`

### 10. `/ip firewall nat print stats`
- **Gdzie:** RouterOS
- **Co robi:** Pokazuje liczniki NAT.
- **Co zobaczyliśmy:** Reguła masquerade miała trafienia.
- **Cofnięcie / uwaga:** `Tylko odczyt.`

### 11. `/ip firewall nat disable 0`
- **Gdzie:** RouterOS
- **Co robi:** Celowo wyłącza regułę NAT.
- **Co zobaczyliśmy:** Usterka laboratoryjna: klient10 stracił Internet.
- **Cofnięcie / uwaga:** `/ip firewall nat enable 0`

### 12. `/ip firewall nat enable 0`
- **Gdzie:** RouterOS
- **Co robi:** Ponownie włącza NAT.
- **Co zobaczyliśmy:** Internet klienta wrócił: 4/4, 0% strat.
- **Cofnięcie / uwaga:** `Jeśli trzeba ponownie wyłączyć: /ip firewall nat disable 0`

### 13. `/ip firewall filter print detail`
- **Gdzie:** RouterOS
- **Co robi:** Pokazuje pełne parametry reguł firewalla.
- **Co zobaczyliśmy:** Potwierdziliśmy forward rules oraz później input rule.
- **Cofnięcie / uwaga:** `Tylko odczyt.`

### 14. `sudo ip netns exec klient20 ping -c 4 192.168.20.1`
- **Gdzie:** Ubuntu
- **Co robi:** Test ruchu do samego routera.
- **Co zobaczyliśmy:** 4/4 - forward nie blokuje ruchu input.
- **Cofnięcie / uwaga:** `Tylko test.`

### 15. `sudo ip netns exec klient20 nc -vz 192.168.20.1 8291`
- **Gdzie:** Ubuntu
- **Co robi:** Test portu WinBox z VLAN20.
- **Co zobaczyliśmy:** Przed regułą input: połączenie succeeded.
- **Cofnięcie / uwaga:** `Tylko test.`

### 16. `/ip firewall filter add chain=input src-address=192.168.20.0/24 protocol=tcp dst-port=8291 action=drop comment="BLOCK WinBox from VLAN20"`
- **Gdzie:** RouterOS
- **Co robi:** Blokuje WinBox do samego routera z sieci gościnnej.
- **Co zobaczyliśmy:** Po dodaniu test nc z VLAN20 kończy się timeoutem.
- **Cofnięcie / uwaga:** `/ip firewall filter remove [find comment="BLOCK WinBox from VLAN20"]`

### 17. `sudo ip netns exec klient20 nc -vz -w 3 192.168.20.1 8291`
- **Gdzie:** Ubuntu
- **Co robi:** Ponowny test WinBox z 3-sekundowym timeoutem.
- **Co zobaczyliśmy:** Timeout - reguła input działa.
- **Cofnięcie / uwaga:** `Tylko test.`


## Testy i dowody

| Test | Przed | Po / wynik | Wniosek |
|---|---|---|---|
| Router -> 1.1.1.1 | - | 4/4, 0% strat | Trasa domyślna i wyjście ether1 działają. |
| klient10 -> 1.1.1.1 | 100% strat bez NAT | 4/4 po masquerade | Brak NAT był przyczyną braku Internetu klienta. |
| klient20 -> 1.1.1.1 | - | 4/4, 0% strat | Jedna reguła masquerade obsługuje również VLAN20. |
| klient20 -> klient10 | - | 100% strat | Reguła forward BLOCK VLAN20->VLAN10 działa. |
| klient10 -> klient20 | - | 4/4, 0% strat | Kierunek odwrotny działa; odpowiedzi wracają dzięki established/related. |
| klient20 -> 192.168.20.1 | - | 4/4, 0% strat | To ruch input do routera, nie forward. |
| klient20 -> WinBox 8291 | succeeded | timeout po regule input | Blokada WinBox z VLAN20 działa. |
| Usterka NAT | NAT disabled -> 100% strat | NAT enabled -> 4/4 | Pełny cykl diagnostyczny zakończony naprawą. |

## Diagnostyka - wzorzec juniora

Zawsze: **objaw -> hipoteza -> test -> interpretacja -> naprawa -> weryfikacja**.

### Klient nie ma Internetu, router ma
- **Hipoteza:** Routing routera działa, ale brak source NAT dla prywatnych podsieci.
- **Test:** Sprawdź /ip route print, ping 1.1.1.1 z routera, /ip firewall nat print, potem ping z klienta.
- **Naprawa:** Dodaj lub włącz poprawną regułę srcnat/masquerade.
- **Weryfikacja:** Ponów ping z klienta i sprawdź /ip firewall nat print stats.

### VLAN20 ma Internet, ale nie powinien wejść do VLAN10
- **Hipoteza:** Trzeba filtrować ruch przechodzący przez router.
- **Test:** Ping klient20 -> 192.168.10.200 oraz /ip firewall filter print stats.
- **Naprawa:** Reguła chain=forward src-address=192.168.20.0/24 dst-address=192.168.10.0/24 action=drop.
- **Weryfikacja:** Ping ma mieć 100% strat, licznik DROP rośnie.

### Gość nie wchodzi do VLAN10, ale może otworzyć WinBox routera
- **Hipoteza:** Blokada jest w forward, a WinBox do routera trafia do input.
- **Test:** nc -vz 192.168.20.1 8291.
- **Naprawa:** Dodaj chain=input drop dla TCP/8291 ze źródła 192.168.20.0/24.
- **Weryfikacja:** nc -vz -w 3 ... powinno timeoutować.

### Po wyłączeniu NAT klient traci Internet
- **Hipoteza:** Celowa usterka - reguła masquerade ma flagę X.
- **Test:** /ip firewall nat print.
- **Naprawa:** Włącz regułę: /ip firewall nat enable 0.
- **Weryfikacja:** Ping 1.1.1.1 z klienta wraca do 4/4.


## Usterka ćwiczeniowa NAT

1. Wyłączyliśmy regułę NAT: `/ip firewall nat disable 0`.
2. `klient10 -> 1.1.1.1` przestał działać: 100% strat.
3. Włączyliśmy regułę ponownie: `/ip firewall nat enable 0`.
4. Ping wrócił: 4/4, 0% strat.
5. Wniosek: routing routera był sprawny, ale brak source NAT uniemożliwiał klientowi poprawną komunikację przez upstream.


## Polityka końcowa labu

- VLAN10 PRACOWNICY -> Internet: **działa**
- VLAN20 GOSCIE -> Internet: **działa**
- VLAN20 -> VLAN10: **zablokowane**
- VLAN10 -> VLAN20: **działa**
- VLAN20 -> WinBox routera TCP/8291: **zablokowane przez input**
- NAT masquerade przez ether1: **włączony**


## Co trzeba jeszcze utrwalić

Na pytania teoretyczne `input` vs `forward` oraz rola `masquerade` odpowiedź nie była jeszcze pewna. To nie zmienia faktu, że konfiguracja i diagnostyka praktyczna zostały wykonane. Na początku Dnia 20 zrobimy 2-minutową powtórkę:
- INPUT = do routera,
- FORWARD = przez router,
- OUTPUT = z routera,
- routing = droga,
- NAT = zmiana adresu,
- firewall = decyzja pozwól/zablokuj.


## Stan nauki do wznowienia

**Dział 5 MikroTik (dni 17-19) zakończony praktycznie.**

Następny etap: **Dzień 20 - VPN: client-to-site i site-to-site; praktyczna konfiguracja WireGuard na Debianie i kliencie, krótkie porównanie z IPsec/OpenVPN.**

Przed startem Dnia 20: krótka powtórka `input/forward/output`, NAT masquerade i różnica routing/NAT/firewall.


## Aktualna dokumentacja producenta wykorzystana do weryfikacji

- MikroTik RouterOS Manual - Firewall Filter: https://manual.mikrotik.com/docs/firewall-and-quality-of-service/firewall/filter/
- MikroTik RouterOS - NAT: https://help.mikrotik.com/docs/spaces/ROS/pages/3211299/NAT
- MikroTik RouterOS Manual - Connection Tracking: https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/
- MikroTik RouterOS Manual - First Time Configuration: https://manual.mikrotik.com/docs/getting-started/first-time-configuration/