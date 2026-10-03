# Dzień 18 - MikroTik RouterOS: VLAN, DHCP, trunk, izolacja i dostęp zarządczy

> Projekt laboratoryjny. RouterOS 7.23.7 (long-term). Materiał opisuje wyłącznie czynności rzeczywiście wykonane i wyniki potwierdzone w laboratorium.

## Stan końcowy laboratorium

```text
Windows 192.168.0.99
        |
Ubuntu/KVM 192.168.0.51
        |
        +-- default / virbr0 ---> ether1 CHR: 192.168.122.75 (WinBox 8291)
        +-- vlan10-lab ---------> ether2 access/PVID10 ---> VLAN10 PRACOWNICY 192.168.10.0/24
        +-- vlan20-lab ---------> ether3 access/PVID20 ---> VLAN20 GOSCIE     192.168.20.0/24
        +-- default ------------> ether4 TRUNK tagged VLAN 10,20,30,40

CHR bridge-vlan:
  VLAN10 -> 192.168.10.1/24, klient10=192.168.10.200
  VLAN20 -> 192.168.20.1/24, klient20=192.168.20.200
  VLAN30 -> 192.168.30.1/24 (KAMERY)
  VLAN40 -> 192.168.40.1/24 (TELEFONY)
```

## Najważniejsze definicje

- **VLAN** - Logiczny podział jednej infrastruktury warstwy 2 na osobne domeny rozgłoszeniowe.
- **Access / untagged** - Port dla urządzenia końcowego; ruch wychodzi zwykle bez taga VLAN. W naszym labie ether2=VLAN10, ether3=VLAN20.
- **Trunk / tagged** - Port przenoszący wiele VLAN-ów. Ramki zachowują tag 802.1Q. U nas ether4 przenosi VLAN 10/20/30/40.
- **PVID** - Port VLAN ID. Określa, do którego VLAN-u przypisać nieotagowany ruch wchodzący na port.
- **Bridge VLAN filtering** - Mechanizm RouterOS, który egzekwuje członkostwo portów w VLAN-ach i tagowanie/odtagowanie.
- **Bridge VLAN table** - Tabela mówiąca, które porty są tagged/untagged dla konkretnych VLAN ID.
- **CPU/bridge port** - Sam interfejs bridge jako port do CPU/routera. Potrzebny m.in. do routingu między VLAN-ami przez interfejsy VLAN.
- **Interfejs VLAN L3** - Logiczny interfejs np. vlan10-pracownicy na bridge-vlan; może mieć adres IP i działać jako brama.
- **DHCP pool** - Zakres adresów IP przeznaczonych do dynamicznego rozdawania klientom.
- **Lease** - Dzierżawa DHCP: czas, przez który klient może używać przydzielonego adresu.
- **Routing między VLAN-ami** - Przekazywanie pakietów między różnymi podsieciami/VLAN-ami przez router.
- **Firewall chain=forward** - Łańcuch dla ruchu przechodzącego przez router, a nie skierowanego do samego routera.
- **connection-state=established** - Pakiet należy do już istniejącego połączenia.
- **connection-state=related** - Pakiet należy do połączenia powiązanego z istniejącym połączeniem.
- **Network namespace** - Izolowany stos sieciowy Linuxa użyty jako lekki klient testowy.
- **veth** - Para wirtualnych interfejsów działająca jak kabel z dwoma końcami.
- **libvirt network** - Wirtualny segment sieciowy KVM/libvirt, np. default, vlan10-lab, vlan20-lab.

## Komendy RouterOS - z wyjaśnieniem i cofnięciem

### 1. `/interface print`
**Gdzie:** RouterOS

**Co robi:** Wyświetla interfejsy i ich stan. Używaliśmy do potwierdzania ether1/ether2/ether3/ether4 oraz ich MAC.

**Cofnięcie / uwaga:** `Brak - polecenie tylko odczytu.`

### 2. `/interface bridge add name=bridge-vlan vlan-filtering=no`
**Gdzie:** RouterOS

**Co robi:** Tworzy bridge do laboratorium VLAN. Filtrowanie VLAN celowo wyłączone na czas budowy konfiguracji.

**Cofnięcie / uwaga:** `/interface bridge remove [find name=bridge-vlan] - tylko gdy bridge nie jest już używany.`

### 3. `/interface bridge port add bridge=bridge-vlan interface=ether2`
**Gdzie:** RouterOS

**Co robi:** Dodaje ether2 jako port bridge.

**Cofnięcie / uwaga:** `/interface bridge port remove [find interface=ether2]`

### 4. `/interface bridge port set [find interface=ether2] pvid=10`
**Gdzie:** RouterOS

**Co robi:** Ustawia PVID 10: nieotagowany ruch wchodzący przez ether2 przypisujemy do VLAN 10.

**Cofnięcie / uwaga:** `/interface bridge port set [find interface=ether2] pvid=1`

### 5. `/interface bridge port add bridge=bridge-vlan interface=ether3`
**Gdzie:** RouterOS

**Co robi:** Dodaje ether3 jako port bridge.

**Cofnięcie / uwaga:** `/interface bridge port remove [find interface=ether3]`

### 6. `/interface bridge port set [find interface=ether3] pvid=20`
**Gdzie:** RouterOS

**Co robi:** Ustawia PVID 20 dla portu gości.

**Cofnięcie / uwaga:** `/interface bridge port set [find interface=ether3] pvid=1`

### 7. `/interface bridge vlan add bridge=bridge-vlan untagged=ether2 vlan-ids=10`
**Gdzie:** RouterOS

**Co robi:** Wpisuje VLAN 10 do tabeli bridge VLAN jako untagged na ether2 (port access).

**Cofnięcie / uwaga:** `/interface bridge vlan remove [find vlan-ids=10] - ostrożnie, jeśli wpis był później rozszerzany.`

### 8. `/interface bridge vlan add bridge=bridge-vlan untagged=ether3 vlan-ids=20`
**Gdzie:** RouterOS

**Co robi:** Wpisuje VLAN 20 jako untagged na ether3.

**Cofnięcie / uwaga:** `/interface bridge vlan remove [find vlan-ids=20] - ostrożnie.`

### 9. `/interface bridge set bridge-vlan vlan-filtering=yes`
**Gdzie:** RouterOS

**Co robi:** Aktywuje świadome filtrowanie VLAN na bridge. Od tej chwili PVID i tabela VLAN są egzekwowane.

**Cofnięcie / uwaga:** `/interface bridge set bridge-vlan vlan-filtering=no`

### 10. `/interface vlan add name=vlan10-pracownicy interface=bridge-vlan vlan-id=10`
**Gdzie:** RouterOS

**Co robi:** Tworzy interfejs L3 VLAN 10 na bridge, potrzebny do adresu bramy/routingu/DHCP.

**Cofnięcie / uwaga:** `/interface vlan remove [find name=vlan10-pracownicy]`

### 11. `/interface vlan add name=vlan20-goscie interface=bridge-vlan vlan-id=20`
**Gdzie:** RouterOS

**Co robi:** Tworzy interfejs L3 VLAN 20.

**Cofnięcie / uwaga:** `/interface vlan remove [find name=vlan20-goscie]`

### 12. `/ip address add address=192.168.10.1/24 interface=vlan10-pracownicy`
**Gdzie:** RouterOS

**Co robi:** Nadaje MikroTikowi adres bramy dla VLAN 10.

**Cofnięcie / uwaga:** `/ip address remove [find address="192.168.10.1/24"]`

### 13. `/ip address add address=192.168.20.1/24 interface=vlan20-goscie`
**Gdzie:** RouterOS

**Co robi:** Nadaje bramę VLAN 20.

**Cofnięcie / uwaga:** `/ip address remove [find address="192.168.20.1/24"]`

### 14. `/ip pool add name=dhcp_pool20 ranges=192.168.20.100-192.168.20.200`
**Gdzie:** RouterOS

**Co robi:** Tworzy pulę adresów DHCP dla gości.

**Cofnięcie / uwaga:** `/ip pool remove [find name=dhcp_pool20]`

### 15. `/ip dhcp-server add name=dhcp20 interface=vlan20-goscie address-pool=dhcp_pool20 lease-time=30m`
**Gdzie:** RouterOS

**Co robi:** Uruchamia serwer DHCP na interfejsie VLAN 20 z pulą dhcp_pool20.

**Cofnięcie / uwaga:** `/ip dhcp-server remove [find name=dhcp20]`

### 16. `/ip dhcp-server network add address=192.168.20.0/24 gateway=192.168.20.1`
**Gdzie:** RouterOS

**Co robi:** Definiuje parametry sieci przekazywane klientom DHCP VLAN 20 (tu: brama).

**Cofnięcie / uwaga:** `/ip dhcp-server network remove [find address="192.168.20.0/24"]`

### 17. `/ip dns print`
**Gdzie:** RouterOS

**Co robi:** Pokazuje statyczne i dynamiczne DNS-y routera. Potwierdziliśmy dynamic-servers=192.168.122.1.

**Cofnięcie / uwaga:** `Brak - odczyt.`

### 18. `/tool sniffer quick interface=ether3 port=67,68`
**Gdzie:** RouterOS

**Co robi:** Podsłuch DHCP na ether3. Potwierdził DISCOVER klienta i odpowiedź serwera DHCP.

**Cofnięcie / uwaga:** `Q kończy sniffer.`

### 19. `/ip firewall filter add chain=forward src-address=192.168.20.0/24 dst-address=192.168.10.0/24 action=drop comment="BLOCK VLAN20 to VLAN10"`
**Gdzie:** RouterOS

**Co robi:** Blokuje nowe i istniejące pakiety z VLAN 20 do VLAN 10. Sama ta reguła blokowała też odpowiedzi na połączenia inicjowane z VLAN 10.

**Cofnięcie / uwaga:** `/ip firewall filter remove [find comment="BLOCK VLAN20 to VLAN10"]`

### 20. `/ip firewall filter add chain=forward connection-state=established,related action=accept comment="ALLOW established related" place-before=0`
**Gdzie:** RouterOS

**Co robi:** Dodaje regułę stateful przed blokadą: odpowiedzi należące do istniejących/related połączeń są przepuszczane.

**Cofnięcie / uwaga:** `/ip firewall filter remove [find comment="ALLOW established related"]`

### 21. `/ip firewall filter print stats`
**Gdzie:** RouterOS

**Co robi:** Pokazuje liczniki pakietów/bytes dla reguł. Potwierdziliśmy trafienia w regułę BLOCK.

**Cofnięcie / uwaga:** `Brak - odczyt.`

### 22. `/interface bridge port add bridge=bridge-vlan interface=ether4`
**Gdzie:** RouterOS

**Co robi:** Dodaje ether4 do bridge jako port przygotowany pod trunk.

**Cofnięcie / uwaga:** `/interface bridge port remove [find interface=ether4]`

### 23. `/interface bridge port set [find interface=ether4] frame-types=admit-only-vlan-tagged`
**Gdzie:** RouterOS

**Co robi:** Na trunku zezwala tylko na ramki tagowane VLAN.

**Cofnięcie / uwaga:** `/interface bridge port set [find interface=ether4] frame-types=admit-all`

### 24. `/interface bridge vlan set [find vlan-ids=10] tagged=bridge-vlan,ether4`
**Gdzie:** RouterOS

**Co robi:** Dodaje ether4 jako tagged member VLAN 10; bridge-vlan pozostaje tagged do CPU/L3.

**Cofnięcie / uwaga:** `Przywrócić poprzednią listę tagged po wcześniejszym sprawdzeniu print detail.`

### 25. `/interface bridge vlan set [find vlan-ids=20] tagged=bridge-vlan,ether4`
**Gdzie:** RouterOS

**Co robi:** Dodaje VLAN 20 do trunku ether4.

**Cofnięcie / uwaga:** `Jak wyżej.`

### 26. `/interface vlan add name=vlan30-kamery interface=bridge-vlan vlan-id=30`
**Gdzie:** RouterOS/WinBox

**Co robi:** Tworzy VLAN 30 dla kamer. Wykonaliśmy w GUI WinBox.

**Cofnięcie / uwaga:** `/interface vlan remove [find name=vlan30-kamery]`

### 27. `/ip address add address=192.168.30.1/24 interface=vlan30-kamery`
**Gdzie:** RouterOS

**Co robi:** Brama dla VLAN 30 KAMERY.

**Cofnięcie / uwaga:** `/ip address remove [find address="192.168.30.1/24"]`

### 28. `/interface bridge vlan add bridge=bridge-vlan vlan-ids=30 tagged=bridge-vlan,ether4`
**Gdzie:** RouterOS

**Co robi:** Dodaje VLAN 30 jako tagged na trunku ether4.

**Cofnięcie / uwaga:** `/interface bridge vlan remove [find vlan-ids=30]`

### 29. `/interface vlan add name=vlan40-telefony interface=bridge-vlan vlan-id=40`
**Gdzie:** RouterOS/WinBox

**Co robi:** Tworzy VLAN 40 dla telefonów. Wykonaliśmy w GUI WinBox.

**Cofnięcie / uwaga:** `/interface vlan remove [find name=vlan40-telefony]`

### 30. `/ip address add address=192.168.40.1/24 interface=vlan40-telefony`
**Gdzie:** RouterOS

**Co robi:** Brama VLAN 40 TELEFONY.

**Cofnięcie / uwaga:** `/ip address remove [find address="192.168.40.1/24"]`

### 31. `/interface bridge vlan add bridge=bridge-vlan vlan-ids=40 tagged=bridge-vlan,ether4`
**Gdzie:** RouterOS

**Co robi:** Dodaje VLAN 40 do trunku ether4.

**Cofnięcie / uwaga:** `/interface bridge vlan remove [find vlan-ids=40]`


## Komendy Ubuntu/KVM/Linux

### 1. `sudo virsh domiflist chr-lab01`
**Gdzie:** Ubuntu/KVM

**Co robi:** Wyświetla karty VM CHR, źródłową sieć libvirt, model i MAC.

**Cofnięcie / uwaga:** `Brak - odczyt.`

### 2. `sudo virsh attach-interface --domain chr-lab01 --type network --source default --model virtio --config --live`
**Gdzie:** Ubuntu/KVM

**Co robi:** Dodaje kartę virtio do CHR na żywo i zapisuje ją w konfiguracji VM.

**Cofnięcie / uwaga:** `Odpiąć właściwy interfejs po MAC/aliasie - najpierw sprawdzić domiflist/dumpxml.`

### 3. `sudo virsh net-list --all`
**Gdzie:** Ubuntu/KVM

**Co robi:** Lista sieci libvirt i ich stan/autostart.

**Cofnięcie / uwaga:** `Brak.`

### 4. `sudo virsh net-define /tmp/vlan10-lab.xml`
**Gdzie:** Ubuntu/KVM

**Co robi:** Definiuje izolowaną sieć libvirt vlan10-lab na podstawie XML.

**Cofnięcie / uwaga:** `sudo virsh net-undefine vlan10-lab - dopiero po zatrzymaniu i odpięciu urządzeń.`

### 5. `sudo virsh net-start vlan10-lab`
**Gdzie:** Ubuntu/KVM

**Co robi:** Uruchamia sieć vlan10-lab.

**Cofnięcie / uwaga:** `sudo virsh net-destroy vlan10-lab`

### 6. `sudo virsh net-autostart vlan10-lab`
**Gdzie:** Ubuntu/KVM

**Co robi:** Włącza start sieci libvirt przy starcie hosta.

**Cofnięcie / uwaga:** `sudo virsh net-autostart vlan10-lab --disable`

### 7. `sudo virsh net-define /tmp/vlan20-lab.xml`
**Gdzie:** Ubuntu/KVM

**Co robi:** Definiuje vlan20-lab.

**Cofnięcie / uwaga:** `sudo virsh net-undefine vlan20-lab`

### 8. `sudo virsh net-start vlan20-lab`
**Gdzie:** Ubuntu/KVM

**Co robi:** Uruchamia vlan20-lab.

**Cofnięcie / uwaga:** `sudo virsh net-destroy vlan20-lab`

### 9. `sudo virsh net-autostart vlan20-lab`
**Gdzie:** Ubuntu/KVM

**Co robi:** Autostart vlan20-lab.

**Cofnięcie / uwaga:** `sudo virsh net-autostart vlan20-lab --disable`

### 10. `sudo virsh dumpxml --inactive chr-lab01 > ~/chr-lab01-before-fix.xml`
**Gdzie:** Ubuntu/KVM

**Co robi:** Kopia definicji VM przed naprawą duplikatu MAC.

**Cofnięcie / uwaga:** `Przywracanie: virsh define plik.xml - tylko świadomie po sprawdzeniu.`

### 11. `sudo virt-xml chr-lab01 --remove-device --network network=default,mac=52:54:00:a5:e1:2e`
**Gdzie:** Ubuntu/KVM

**Co robi:** Usunęliśmy offline stary interfejs z sieci default, który dublował MAC interfejsu VLAN20.

**Cofnięcie / uwaga:** `Mamy kopię chr-lab01-before-fix.xml.`

### 12. `sudo ip netns add klient10`
**Gdzie:** Ubuntu/Linux

**Co robi:** Tworzy namespace sieciowy klient10 - lekki klient testowy.

**Cofnięcie / uwaga:** `sudo ip netns del klient10`

### 13. `sudo ip link add veth10-host type veth peer name veth10-client`
**Gdzie:** Ubuntu/Linux

**Co robi:** Tworzy parę veth - wirtualny kabel z dwoma końcami.

**Cofnięcie / uwaga:** `sudo ip link del veth10-host`

### 14. `sudo ip link set veth10-host master virbr10`
**Gdzie:** Ubuntu/Linux

**Co robi:** Podłącza hostowy koniec veth do bridge libvirt VLAN 10.

**Cofnięcie / uwaga:** `sudo ip link set veth10-host nomaster`

### 15. `sudo ip link set veth10-client netns klient10`
**Gdzie:** Ubuntu/Linux

**Co robi:** Przenosi drugi koniec kabla do namespace klient10.

**Cofnięcie / uwaga:** `Usunięcie namespace usuwa interfejs znajdujący się wewnątrz.`

### 16. `sudo ip netns exec klient10 dhclient -v veth10-client`
**Gdzie:** Ubuntu/Linux

**Co robi:** Klient DHCP w namespace. Otrzymaliśmy 192.168.10.200/24.

**Cofnięcie / uwaga:** `sudo ip netns exec klient10 dhclient -r veth10-client`

### 17. `sudo ip netns exec klient10 ip route`
**Gdzie:** Ubuntu/Linux

**Co robi:** Sprawdza routing klienta10. Potwierdziliśmy default via 192.168.10.1.

**Cofnięcie / uwaga:** `Brak.`

### 18. `sudo ip netns exec klient10 ping -c 4 192.168.10.1`
**Gdzie:** Ubuntu/Linux

**Co robi:** Test do bramy VLAN 10: 4/4, 0% strat.

**Cofnięcie / uwaga:** `Brak.`

### 19. `sudo ip netns add klient20`
**Gdzie:** Ubuntu/Linux

**Co robi:** Tworzy klient20.

**Cofnięcie / uwaga:** `sudo ip netns del klient20`

### 20. `sudo ip netns exec klient20 dhclient -v veth20-client`
**Gdzie:** Ubuntu/Linux

**Co robi:** Pobiera adres z DHCP VLAN 20. Otrzymaliśmy 192.168.20.200/24.

**Cofnięcie / uwaga:** `sudo ip netns exec klient20 dhclient -r veth20-client`

### 21. `sudo ip netns exec klient20 ping -c 4 192.168.20.1`
**Gdzie:** Ubuntu/Linux

**Co robi:** Test do bramy VLAN 20: 4/4, 0% strat.

**Cofnięcie / uwaga:** `Brak.`

### 22. `sudo ip netns exec klient10 ping -c 4 192.168.20.200`
**Gdzie:** Ubuntu/Linux

**Co robi:** Test routingu między VLAN-ami. Przed firewallem działał; po stateful firewall działa z VLAN10 do VLAN20.

**Cofnięcie / uwaga:** `Brak.`

### 23. `sudo ip netns exec klient20 ping -c 4 192.168.10.200`
**Gdzie:** Ubuntu/Linux

**Co robi:** Test izolacji gości. Po regułach firewalla: 100% strat.

**Cofnięcie / uwaga:** `Brak.`

### 24. `nc -vz 192.168.122.75 8291`
**Gdzie:** Ubuntu/Linux

**Co robi:** Potwierdza, że usługa WinBox 8291 jest osiągalna z hosta Ubuntu.

**Cofnięcie / uwaga:** `Brak.`

### 25. `sysctl net.ipv4.ip_forward`
**Gdzie:** Ubuntu/Linux

**Co robi:** Sprawdza forwarding IPv4; wynik był 1.

**Cofnięcie / uwaga:** `Brak.`

### 26. `sudo nft list ruleset`
**Gdzie:** Ubuntu/Linux

**Co robi:** Pokazał reguły libvirt. LIBVIRT_FWI odrzucał nowe połączenia do virbr0.

**Cofnięcie / uwaga:** `Brak - odczyt.`

### 27. `sudo iptables -I FORWARD 1 -i eno1 -o virbr0 -s 192.168.0.99 -d 192.168.122.75 -p tcp --dport 8291 -j ACCEPT`
**Gdzie:** Ubuntu/Linux

**Co robi:** Tymczasowo dopuszcza tylko Windows 192.168.0.99 do WinBox 192.168.122.75:8291 przez host Ubuntu.

**Cofnięcie / uwaga:** `sudo iptables -D FORWARD -i eno1 -o virbr0 -s 192.168.0.99 -d 192.168.122.75 -p tcp --dport 8291 -j ACCEPT`


## Komendy Windows

### 1. `Test-NetConnection 192.168.122.75 -Port 8291`
**Gdzie:** Windows PowerShell

**Co robi:** Test TCP do WinBoxa. Najpierw False, po korekcie routingu/forwardingu True.

**Cofnięcie / uwaga:** `Brak.`

### 2. `route add 192.168.122.0 mask 255.255.255.0 192.168.0.51`
**Gdzie:** Windows PowerShell (Administrator)

**Co robi:** Dodaje tymczasową trasę do sieci KVM 192.168.122.0/24 przez Ubuntu 192.168.0.51.

**Cofnięcie / uwaga:** `route delete 192.168.122.0`


## Testy i dowody wykonania

| Test | Wynik | Status |
|---|---|---|
| VLAN 10 DHCP | klient10 dostał 192.168.10.200/24 | ZALICZONE |
| VLAN 10 brama | ping 192.168.10.1: 4/4, 0% strat | ZALICZONE |
| VLAN 20 DHCP | klient20 dostał 192.168.20.200/24 | ZALICZONE |
| VLAN 20 brama | ping 192.168.20.1: 4/4, 0% strat | ZALICZONE |
| Routing przed filtrem | VLAN10↔VLAN20 działał w obie strony | POTWIERDZONE |
| Izolacja VLAN20→VLAN10 | ping 4 wysłane, 0 odebrane, 100% strat | ZALICZONE |
| Stateful VLAN10→VLAN20 | po ALLOW established,related: 4/4, 0% strat | ZALICZONE |
| WinBox z Windows | Test-NetConnection ... -Port 8291 => TcpTestSucceeded: True | ZALICZONE |
| Trunk ether4 | VLAN 10,20,30,40 jako tagged=bridge-vlan,ether4 | ZALICZONE |

## Usterki przećwiczone

### WinBox timeout z Windows
- **Przyczyna:** Windows nie miał poprawnej ścieżki do 192.168.122.0/24, a libvirt odrzucał nowe połączenia do virbr0.
- **Naprawa/weryfikacja:** route na Windows + wąska reguła FORWARD na Ubuntu; Test-NetConnection=True.

### Duplikat MAC po przepięciu ether3
- **Przyczyna:** Nowy interfejs vlan20-lab został dołączony zanim stary default został poprawnie usunięty.
- **Naprawa/weryfikacja:** Kopia XML, wyłączenie VM, virt-xml --remove-device po network+MAC, restart i domiflist.

### DHCP VLAN20 początkowo bez widocznego OFFER w terminalu
- **Przyczyna:** Klient ponawiał DISCOVER; sniffer RouterOS pokazał DISCOVER i odpowiedź 192.168.20.1→192.168.20.200.
- **Naprawa/weryfikacja:** Sprawdzenie ip addr wykazało, że klient ostatecznie był bound do 192.168.20.200.

### Komendy RouterOS wpisane w Bash
- **Przyczyna:** Polecenia /interface lub /ip uruchamiane w Ubuntu dają 'No such file or directory'.
- **Naprawa/weryfikacja:** Najpierw wejść do CHR przez virsh console albo użyć WinBox Terminal.

### Wklejanie promptu/wyniku jako komendy
- **Przyczyna:** RouterOS próbował wykonać '[admin@CHR] >', 'Flags:' lub fragmenty outputu.
- **Naprawa/weryfikacja:** Wpisywać tylko właściwą komendę zaczynającą się np. od /interface.


## Co trzeba powtórzyć przed Dniem 19

- trunk != port internetu; trunk przenosi wiele VLAN-ów jako tagged.
- PVID nie nadaje adresu IP; przypisuje nieotagowany ruch wejściowy do VLAN-u.
- VLAN rozdziela domeny L2, routing łączy podsieci L3, firewall decyduje kto może przejść.
- `established,related` przepuszcza odpowiedzi/powiązany ruch; nie tworzy VLAN-ów.


## Uwagi o trwałości konfiguracji

- RouterOS: konfiguracja VLAN/DHCP/firewall została zachowana po restarcie CHR.
- Sieci `vlan10-lab` i `vlan20-lab` mają autostart.
- Namespace `klient10/klient20` i pary veth są elementami tymczasowymi hosta Linux i po restarcie mogą wymagać odtworzenia.
- `route add` w Windows bez `-p` jest trasą tymczasową.
- Reguła `iptables -I FORWARD ...8291` została dodana runtime; przed traktowaniem jej jako stałej trzeba świadomie zapisać ją w trwałej konfiguracji firewalla hosta.


## Źródła producentów

- MikroTik RouterOS Manual - Bridging and Switching: https://manual.mikrotik.com/docs/bridging-and-switching/
- MikroTik RouterOS Documentation - DHCP: https://help.mikrotik.com/docs/spaces/ROS/pages/24805500/DHCP
- MikroTik RouterOS Manual - Connection tracking: https://manual.mikrotik.com/docs/firewall-and-quality-of-service/connection-tracking/
- MikroTik RouterOS Manual - Firewall filter: https://manual.mikrotik.com/docs/cli-reference/ip/firewall/filter/
- libvirt Networking: https://libvirt.org/formatnetwork.html