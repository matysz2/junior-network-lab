# Dzień 12 - WLAN, DHCP, sieć gościnna, ACL i DNS

## Cel dnia

Celem Dnia 12 było praktyczne zbudowanie dwóch sieci Wi-Fi w Cisco
Packet Tracerze i rozdzielenie ich logicznie:

-   `Firma_Pracownicy` -\> VLAN 10 -\> `192.168.10.0/24`
-   `Firma_Goscie` -\> VLAN 20 -\> `192.168.20.0/24`

Następnie skonfigurowaliśmy DHCP, DNS oraz ACL tak, aby goście mogli
korzystać z własnej sieci, ale nie mieli dostępu do sieci pracowników.

> Ważne: dokument opisuje tylko rzeczy rzeczywiście wykonane podczas
> laboratorium Dnia 12.

------------------------------------------------------------------------

## 1. Topologia po Dniu 12

``` text
                    Router1
               G0/0 - trunk 802.1Q
                 /             \
        G0/0.10                   G0/0.20
     192.168.10.1              192.168.20.1
        VLAN 10                   VLAN 20
           |                         |
        Switch0                   Switch0
        Fa0/3                     Fa0/4
           |                         |
    Access Point0              Access Point2
 Firma_Pracownicy              Firma_Goscie
           |                         |
      192.168.10.x               192.168.20.x
```

Switch0 jest połączony z Router1 przez `Gig0/1`, pracujący jako trunk
dla VLAN 10 i 20.

------------------------------------------------------------------------

## 2. Najważniejsza idea - SSID, VLAN i podsieć to różne rzeczy

**SSID** to nazwa sieci bezprzewodowej widoczna dla użytkownika, np.
`Firma_Goscie`.

**VLAN** logicznie oddziela ruch na przełączniku. W laboratorium: - VLAN
10 = PRACOWNICY - VLAN 20 = GOSCIE

**Podsieć IP** określa adresację warstwy 3: - VLAN 10 -\>
`192.168.10.0/24` - VLAN 20 -\> `192.168.20.0/24`

Sam VLAN nie jest firewallem. Router posiada interfejsy w obu VLAN-ach,
dlatego bez ACL może routować ruch pomiędzy nimi.

------------------------------------------------------------------------

## 3. Pierwszy problem - laptop dostał 169.254.x.x

Laptop był połączony z Wi-Fi, ale `ipconfig` pokazywał adres z zakresu
`169.254.x.x`.

To adres APIPA. Jest bardzo mocną wskazówką, że klient nie otrzymał
poprawnej konfiguracji IPv4 z DHCP.

### Diagnostyka wykonana krok po kroku

1.  Sprawdziliśmy, czy laptop jest ustawiony na DHCP.
2.  Sprawdziliśmy `show running-config` na Router1.
3.  Okazało się, że DHCP nie było skonfigurowane.
4.  Utworzyliśmy pulę DHCP `PRACOWNICY`.
5.  DHCP nadal nie działało.
6.  `show vlan brief` i `show interfaces status` pokazały, że Access
    Point był na `Fa0/3`, ale port znajdował się w VLAN 1.
7.  Potwierdziliśmy fizyczne połączenie AP -\> `Fa0/3`.
8.  Przenieśliśmy `Fa0/3` do VLAN 10.
9.  `show interfaces trunk` potwierdziło, że VLAN 10 i 20 przechodzą
    przez trunk.
10. `show cdp neighbors` potwierdziło połączenie `Switch0 Gig0/1` -\>
    `Router1 Gig0/0`.
11. Test statycznym adresem `192.168.10.50/24` potwierdził komunikację z
    bramą `192.168.10.1`.
12. Ponowiliśmy DHCP.
13. Router przydzielił laptopowi `192.168.10.21`.

### Wniosek

Nie zmieniamy losowo ustawień. Najpierw ustalamy, na którym etapie
kończy się poprawna komunikacja.

------------------------------------------------------------------------

## 4. DHCP dla pracowników

Konfiguracja wykonana na Router1:

``` text
configure terminal
ip dhcp pool PRACOWNICY
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 exit
ip dhcp excluded-address 192.168.10.1 192.168.10.20
```

Znaczenie: - `ip dhcp pool PRACOWNICY` - tworzy pulę DHCP. -
`network ...` - wskazuje sieć, z której będą przydzielane adresy. -
`default-router 192.168.10.1` - przekazuje klientowi adres bramy. -
`excluded-address .1 .20` - rezerwuje pierwsze adresy, aby DHCP ich nie
przydzielał.

Weryfikacja:

``` text
show ip dhcp pool
show ip dhcp binding
```

Zaobserwowana dzierżawa:

``` text
192.168.10.21    0001.6439.08D9    Automatic
```

------------------------------------------------------------------------

## 5. Port Access Pointa pracowników

Port `Fa0/3` został przypisany do VLAN 10:

``` text
configure terminal
interface fastEthernet0/3
 switchport mode access
 switchport access vlan 10
```

`switchport mode access` oznacza, że port obsługuje urządzenie końcowe
należące do jednego VLAN-u.

------------------------------------------------------------------------

## 6. Weryfikacja trunku i połączenia z routerem

Użyliśmy:

``` text
show interfaces trunk
```

Potwierdziliśmy: - `Gig0/1` = trunk - VLAN 10 i VLAN 20 są dozwolone -
VLAN 10 i VLAN 20 są aktywne i forwarding

Następnie:

``` text
show cdp neighbors
```

potwierdziło:

``` text
Switch0 Gig0/1 -> Router1 Gig0/0
```

To ważna zasada: przed zmianą konfiguracji warto potwierdzić, gdzie
naprawdę prowadzi port.

------------------------------------------------------------------------

## 7. Test statycznego adresu

Na laptopie ustawiliśmy tymczasowo:

``` text
IP:      192.168.10.50
Maska:   255.255.255.0
Brama:   192.168.10.1
```

Następnie:

``` text
ping 192.168.10.1
```

Wynik: `4/4`, `0% loss`.

Wniosek: Wi-Fi, AP, VLAN 10, trunk i router działały. Problem był
związany z DHCP, a nie z podstawową komunikacją.

------------------------------------------------------------------------

## 8. Sieć gościnna

Dodaliśmy drugi Access Point (`Access Point2`) i podłączyliśmy go do
`Switch0 Fa0/4`.

Port został przypisany do VLAN 20:

``` text
configure terminal
interface fastEthernet0/4
 switchport mode access
 switchport access vlan 20
```

Na Access Point2 skonfigurowaliśmy: - SSID: `Firma_Goscie` -
zabezpieczenie: WPA2-PSK - hasło laboratoryjne zostało ustawione podczas
ćwiczenia

W dokumentacji portfolio nie należy publikować prawdziwych haseł
produkcyjnych.

------------------------------------------------------------------------

## 9. DHCP dla gości

Na Router1:

``` text
configure terminal
ip dhcp pool GOSCIE
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 exit
ip dhcp excluded-address 192.168.20.1 192.168.20.20
```

Laptop po połączeniu z `Firma_Goscie` dostał poprawny adres z sieci VLAN
20.

Zaobserwowane adresy podczas testów: `192.168.20.22`, a po późniejszym
odnowieniu `192.168.20.21`.

------------------------------------------------------------------------

## 10. Dlaczego VLAN nie wystarczył do izolacji gości?

Przed ACL laptop gościa miał:

``` text
IP:    192.168.20.22
Brama: 192.168.20.1
```

i mógł wykonać:

``` text
ping 192.168.10.1
```

z wynikiem `4/4`.

Dlaczego? Router posiada: - `G0/0.10` -\> `192.168.10.1` - `G0/0.20` -\>
`192.168.20.1`

Zna więc obie sieci i routuje między nimi.

**VLAN segmentuje sieć warstwy 2. ACL kontroluje, jaki ruch warstwy 3
może przejść przez router.**

------------------------------------------------------------------------

## 11. ACL GOSCIE

Utworzyliśmy rozszerzoną ACL:

``` text
configure terminal
ip access-list extended GOSCIE
 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 permit ip 192.168.20.0 0.0.0.255 any
```

Znaczenie pierwszej reguły:

``` text
VLAN 20 192.168.20.0/24 -> VLAN 10 192.168.10.0/24 = DENY
```

Druga reguła pozwala na pozostały ruch z VLAN 20.

### Wildcard mask

Dla `/24`:

``` text
maska podsieci: 255.255.255.0
wildcard:       0.0.0.255
```

------------------------------------------------------------------------

## 12. Niejawne deny any

ACL Cisco ma na końcu niewidoczną zasadę:

``` text
deny any
```

Dlatego gdybyśmy utworzyli tylko:

``` text
deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
```

pozostały ruch, który nie pasowałby do żadnego `permit`, również
zostałby odrzucony.

ACL jest sprawdzana od góry. Pierwsze dopasowanie kończy analizę
pakietu.

------------------------------------------------------------------------

## 13. Przypięcie ACL do VLAN 20

ACL została przypięta do subinterfejsu gości:

``` text
interface gigabitEthernet0/0.20
 ip access-group GOSCIE in
```

`in` oznacza, że ACL analizuje pakiety **wchodzące do routera** przez
`G0/0.20`.

Nie oznacza "Internet".

Schemat:

``` text
Laptop gościa
      |
      v
G0/0.20 [IN]
      |
   Router1
```

------------------------------------------------------------------------

## 14. Weryfikacja ACL

Przed ACL:

``` text
192.168.20.22 -> ping 192.168.10.1 -> działa
```

Po ACL:

``` text
Reply from 192.168.20.1: Destination host unreachable.
Sent = 4, Received = 0, Lost = 4
```

Jednocześnie:

``` text
ping 192.168.20.1
```

działał `4/4`.

To udowodniło, że laptop nie stracił całej sieci. Zablokowany był
konkretnie ruch do VLAN 10.

------------------------------------------------------------------------

## 15. Liczniki ACL

Polecenie:

``` text
show access-lists
```

po pierwszym teście pokazało:

``` text
10 deny ... (4 match(es))
20 permit ... (4 match(es))
```

Licznik `matches` pokazuje, ile pakietów dopasowało się do danej reguły.

Jeśli klient wykonuje test, a licznik pozostaje `0`, należy sprawdzić
m.in.: - czy ruch przechodzi przez ten interfejs, - czy ACL jest
przypięta, - czy kierunek `in/out` jest poprawny, - czy adres źródłowy i
docelowy pasują do reguły.

------------------------------------------------------------------------

## 16. DNS przez DHCP

Na laptopie gościa początkowo widzieliśmy:

``` text
DNS Server: 0.0.0.0
```

Do puli `GOSCIE` dodaliśmy:

``` text
ip dhcp pool GOSCIE
 dns-server 8.8.8.8
```

Laboratoryjnie użyliśmy `8.8.8.8`. W sieci firmowej adres DNS dobiera
się zgodnie z projektem organizacji.

------------------------------------------------------------------------

## 17. Drugi problem - ACL zablokowała DHCP

Po aktywowaniu ACL i próbie odnowienia DHCP laptop dostał:

``` text
DHCP failed. APIPA is being used.
169.254.x.x
```

To bardzo ważny przypadek.

Klient, który jeszcze nie ma adresu IPv4, rozpoczyna DHCP bez adresu z
sieci `192.168.20.0/24`. Nasza reguła:

``` text
permit ip 192.168.20.0 0.0.0.255 any
```

nie obejmowała początkowego ruchu DHCP. Na końcu ACL działało niejawne
`deny any`.

### Naprawa

Dodaliśmy regułę DHCP z numerem 5:

``` text
ip access-list extended GOSCIE
 5 permit udp any eq bootpc any eq bootps
```

`bootpc` odpowiada portowi UDP klienta DHCP (68), a `bootps` portowi UDP
serwera DHCP (67).

Numer `5` umieścił regułę przed istniejącymi `10` i `20`.

Po zmianie:

``` text
5  permit DHCP
10 deny VLAN20 -> VLAN10
20 permit VLAN20 -> any
```

------------------------------------------------------------------------

## 18. Końcowa weryfikacja

Po poprawce DHCP laptop otrzymał:

``` text
IPv4:   192.168.20.21
Maska:  255.255.255.0
Brama:  192.168.20.1
DNS:    8.8.8.8
```

ACL nadal blokowała:

``` text
ping 192.168.10.1 -> 100% loss
```

Końcowe liczniki:

``` text
5 permit udp any eq bootpc any eq bootps (3 match(es))
10 deny ip 192.168.20.0 ... 192.168.10.0 ... (8 match(es))
20 permit ip 192.168.20.0 ... any (4 match(es))
```

Interpretacja: - reguła 5 faktycznie przepuszcza DHCP, - licznik deny
wzrósł z 4 do 8 po kolejnych czterech pingach do VLAN 10, - reguła
permit przepuszcza dozwolony ruch.

------------------------------------------------------------------------

## 19. Schemat diagnostyczny do zapamiętania

``` text
Klient zgłasza problem
        |
        v
ipconfig
        |
        +-- 169.254.x.x?
        |       |
        |       +--> DHCP / VLAN / droga do DHCP / ACL
        |
        +-- poprawne IP i brama?
                |
                v
          ping własnej bramy
                |
        +-------+-------+
        |               |
      NIE              TAK
        |               |
 AP/VLAN/L2/droga       v
                 ping adresu IP, np. 8.8.8.8
                        |
                 +------+------+
                 |             |
                NIE           TAK
                 |             |
          routing/ACL/WAN      v
                        nazwa nie działa?
                              |
                              +--> DNS
```

------------------------------------------------------------------------

## 20. Komendy Dnia 12 - ściąga

### Tryby Cisco

``` text
enable
configure terminal
exit
end
```

### VLAN / porty / trunk

``` text
show vlan brief
show interfaces status
show interfaces trunk
show cdp neighbors

interface fastEthernet0/3
 switchport mode access
 switchport access vlan 10

interface fastEthernet0/4
 switchport mode access
 switchport access vlan 20
```

### DHCP

``` text
ip dhcp pool PRACOWNICY
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1

ip dhcp pool GOSCIE
network 192.168.20.0 255.255.255.0
default-router 192.168.20.1
dns-server 8.8.8.8

ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20

show ip dhcp pool
show ip dhcp binding
```

### ACL

``` text
ip access-list extended GOSCIE
5 permit udp any eq bootpc any eq bootps
10 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
20 permit ip 192.168.20.0 0.0.0.255 any

interface gigabitEthernet0/0.20
ip access-group GOSCIE in

show access-lists
```

### Klient

``` text
ipconfig
ping 192.168.20.1
ping 192.168.10.1
```

------------------------------------------------------------------------

## 21. Typowe objawy i pierwsze podejrzenie

  -----------------------------------------------------------------------
  Objaw                               Pierwsze podejrzenie
  ----------------------------------- -----------------------------------
  `169.254.x.x`                       DHCP / droga do DHCP / ACL

  `0.0.0.0` jako brama                brak pełnej konfiguracji IP

  ping własnej bramy nie działa       Wi-Fi/AP/VLAN/L2/droga do bramy

  ping IP działa, nazwa nie           DNS

  poprawne IP i brama, ale konkretna  routing lub ACL
  sieć nie działa                     

  `show access-lists`: deny rośnie    ACL faktycznie blokuje ruch

  `show access-lists`: 0 matches mimo ruch nie trafia w regułę / zły
  testu                               interfejs / kierunek / adresy
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 22. Co trzeba umieć wyjaśnić na rozmowie juniora

1.  `169.254.x.x` sugeruje brak poprawnej odpowiedzi DHCP.
2.  VLAN nie jest automatycznie firewallem.
3.  Routing między VLAN-ami pozwala różnym VLAN-om się komunikować.
4.  ACL może kontrolować ten ruch.
5.  ACL jest sprawdzana od góry, do pierwszego dopasowania.
6.  Na końcu ACL znajduje się niejawne `deny any`.
7.  `in` i `out` opisują kierunek względem interfejsu.
8.  Liczniki `matches` pomagają potwierdzić, czy ruch trafia w regułę.
9.  DNS tłumaczy nazwy na adresy IP.
10. Po każdej zmianie należy sprawdzić zarówno funkcję, którą chcieliśmy
    zmienić, jak i usługi, które miały pozostać sprawne.

------------------------------------------------------------------------

## 23. Status Dnia 12

### Wykonane praktycznie

-   dwa SSID w Packet Tracerze,
-   WPA2-PSK,
-   Access Point dla pracowników w VLAN 10,
-   Access Point dla gości w VLAN 20,
-   DHCP dla VLAN 10 i VLAN 20,
-   wykluczenie adresów DHCP,
-   test statycznego IPv4,
-   diagnostyka APIPA `169.254.x.x`,
-   weryfikacja VLAN i trunku,
-   CDP do potwierdzenia połączenia switch-router,
-   routing między VLAN-ami,
-   rozszerzona ACL `GOSCIE`,
-   blokada VLAN 20 -\> VLAN 10,
-   test kierunku `in`,
-   analiza liczników ACL,
-   wykrycie, że ACL zablokowała DHCP,
-   wyjątek ACL dla DHCP UDP 68 -\> 67,
-   DNS `8.8.8.8` przekazywany przez DHCP,
-   końcowa weryfikacja DHCP i izolacji gości.

### Do dalszego utrwalania

-   DHCP vs DNS,
-   VLAN vs ACL,
-   `in` vs `out`,
-   kolejność reguł ACL,
-   wildcard mask,
-   diagnostyka na podstawie objawu, a nie zgadywania.

------------------------------------------------------------------------

## 24. Następny krok

Na początku kolejnej sesji warto zrobić krótką powtórkę bez
podpowiedzi: 1. APIPA `169.254.x.x`, 2. DNS vs DHCP, 3. kolejność ACL,
4. `show access-lists`, 5. `show ip dhcp binding`.

Potem przejść do kolejnego etapu programu 28-dniowego.
