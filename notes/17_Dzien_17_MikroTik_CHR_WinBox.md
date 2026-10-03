# Dzien 17 - MikroTik CHR / RouterOS / WinBox

## Status laboratorium
Wykonano praktycznie 2026-09-30. CHR `chr-lab01` dziala jako VM KVM/libvirt. Potwierdzony adres CHR: `192.168.122.75/24`, brama: `192.168.122.1`, interfejs: `ether1`.

## Topologia
```text
Windows 192.168.0.99
        |
Ubuntu 192.168.0.51 (eno1)
        |
virbr0 192.168.122.1/24
        |
CHR 192.168.122.75/24
```

## 1. KVM/libvirt
```bash
ls -l /dev/kvm
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients -y
virsh --version
sudo virsh list --all
```

## 2. CHR
```bash
mkdir -p ~/mikrotik-chr
cd ~/mikrotik-chr
wget https://download.mikrotik.com/routeros/7.23.7/chr-7.23.7.img.zip
unzip chr-7.23.7.img.zip
qemu-img info chr-7.23.7.img
cp chr-7.23.7.img chr-lab01.img
```
Oryginalny obraz pozostawiono bez zmian, a VM pracuje na kopii.

## 3. Siec libvirt
```bash
sudo virsh net-list --all
sudo virsh net-dumpxml default
```
`virbr0 = 192.168.122.1/24`, DHCP/NAT libvirt.

## 4. VM
```bash
sudo apt install virtinst -y
sudo virt-install --name chr-lab01 --memory 256 --vcpus 1 \
  --disk path=/home/mateusz-czarnik/mikrotik-chr/chr-lab01.img,format=raw,bus=virtio \
  --network network=default,model=virtio --import \
  --osinfo detect=on,require=off --graphics none --console pty,target_type=serial
```

## 5. RouterOS - pierwsze testy
```text
/interface print
/ip address print
/ip dhcp-client print
/ip route print
/ping 1.1.1.1 count=3
/ping mikrotik.com count=3
```
DHCP client byl `bound`; adres dynamiczny `192.168.122.75/24`; default route przez `192.168.122.1`.

## 6. Safe Mode
`Ctrl+X` wlacza Safe Mode. W labie potwierdzono prompt `<SAFE>` i wyjscie bez zmian.

## 7. Backup i eksport
```text
/system backup save name=day17-before-changes
/export file=day17-config
/file print
```
- `.backup` - binarny backup do odtworzenia.
- `.rsc` - tekstowy eksport, czytelny i dobry do dokumentacji.

## 8. Rzeczywista diagnostyka Windows -> CHR
1. Windows nie pingowal `192.168.122.75`.
2. `route print` pokazal brak trasy `192.168.122.0/24`.
3. Dodano tymczasowa trase przez Ubuntu:
```powershell
route add 192.168.122.0 mask 255.255.255.0 192.168.0.51
```
4. Ubuntu: `net.ipv4.ip_forward = 1`.
5. `LIBVIRT_FWI` odrzucal ruch `REJECT`.
6. `ip neigh show 192.168.122.75` pokazal `FAILED`; `virsh list --all` wykazal `chr-lab01 shut off`.
7. VM uruchomiono `sudo virsh start chr-lab01`.
8. Po restarcie libvirt nasz ACCEPT znalazl sie za REJECT. Usunieto zla pozycje i wstawiono ACCEPT przed REJECT:
```bash
sudo iptables -I LIBVIRT_FWI 2 -s 192.168.0.99 -d 192.168.122.0/24 -o virbr0 -j ACCEPT
```
9. Windows ping: 4/4 odpowiedzi z `192.168.122.75`.

Rollback trasy Windows:
```powershell
route delete 192.168.122.0
```

## 9. Konsola CHR
```bash
sudo virsh console chr-lab01
```
Polecenie wykonujemy na Ubuntu. `Ctrl+]` odlacza konsole bez wylaczania VM.

## 10. WinBox
`/ip service print` potwierdzil `winbox` TCP `8291`. Windows polaczyl sie WinBoxem do `192.168.122.75` jako `admin`.

## 11. WinBox - znaczenie ekranow
- Interfaces: `ether1`, `lo`, flaga Running.
- IP -> DHCP Client: `bound` = klient dostal konfiguracje.
- IP -> Addresses: `D` = Dynamic.
- IP -> Routes: `0.0.0.0/0 -> 192.168.122.1`.
- Bridge: obecnie pusty; bridge ~= switch L2. Nie dodawalismy `ether1` do bridge.

## 12. Do powtorki
- Pierwsza brama CHR: `192.168.122.1`.
- Bridge ~= switch; routing laczy rozne sieci IP.
- DHCP Client prosi, DHCP Server rozdaje.
- Kolejnosc regul firewall ma znaczenie.
