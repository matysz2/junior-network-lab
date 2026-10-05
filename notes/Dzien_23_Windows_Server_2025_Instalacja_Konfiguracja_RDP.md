# Dzien 23 - Windows Server 2025: instalacja, siec, DNS i RDP

## Cel

Zainstalowanie Windows Server 2025 jako maszyny wirtualnej na Ubuntu/KVM i przygotowanie go do Dnia 24 (Active Directory).

## Stan koncowy

| Element | Stan |
|---|---|
| VM | `windows-server-2025` |
| System | Windows Server 2025 Standard Evaluation (Desktop Experience) |
| Hostname | `SRV-DC01` |
| IPv4 | `192.168.122.24/24` |
| Brama | `192.168.122.1` |
| DNS na Dniu 23 | `192.168.122.1` |
| MAC VM | `52:54:00:bf:04:63` |
| RDP | dziala, TCP 3389 potwierdzony |
| Klient administracyjny | Windows `192.168.0.99` |
| Host Ubuntu | `192.168.0.51` |

> Prywatne haslo Administratora nie jest zapisane w dokumentacji.

---

## 1. Kopiowanie ISO z Windows do Ubuntu

PowerShell na Windows:

```powershell
scp "H:\26100.32230.260111-0550.lt_release_svc_refresh_SERVER_EVAL_x64FRE_en-us.iso" mateusz-czarnik@192.168.0.51:/home/mateusz-czarnik/
```

Weryfikacja na Ubuntu:

```bash
ls -lh /home/mateusz-czarnik/26100.32230.260111-0550.lt_release_svc_refresh_SERVER_EVAL_x64FRE_en-us.iso
```

Rzeczywisty wynik kopiowania: `100% 7775MB`.

---

## 2. Zasoby hosta

```bash
df -h /
free -h
which virt-install
```

W labie:
- ok. 375 GB wolnego miejsca;
- 7,5 GiB RAM;
- `virt-install` dostepny w `/usr/bin/virt-install`.

VM:
- 3072 MB RAM,
- 2 vCPU,
- 60 GB qcow2.

---

## 3. Dysk VM

```bash
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/windows-server-2025.qcow2 60G
sudo qemu-img info /var/lib/libvirt/images/windows-server-2025.qcow2
```

Wynik:
- virtual size: 60 GiB,
- poczatkowy disk size: ok. 196 KiB,
- `corrupt: false`.

---

## 4. Siec libvirt

```bash
virsh net-list --all
```

Wybrano siec `default` - NAT libvirt, `192.168.122.0/24`.

---

## 5. Utworzenie VM

```bash
sudo virt-install \
  --name windows-server-2025 \
  --memory 3072 \
  --vcpus 2 \
  --cpu host \
  --disk path=/var/lib/libvirt/images/windows-server-2025.qcow2,format=qcow2,bus=sata \
  --cdrom /home/mateusz-czarnik/26100.32230.260111-0550.lt_release_svc_refresh_SERVER_EVAL_x64FRE_en-us.iso \
  --network network=default,model=e1000 \
  --graphics spice \
  --video qxl \
  --os-variant win11 \
  --noautoconsole
```

---

## 6. SPICE przez SSH

Ubuntu:

```bash
virsh domdisplay windows-server-2025
```

Wynik:

```text
spice://127.0.0.1:5900
```

Windows - tunel:

```powershell
ssh -N -L 5901:127.0.0.1:5900 mateusz-czarnik@192.168.0.51
```

Test:

```powershell
Test-NetConnection 127.0.0.1 -Port 5901
```

Remote Viewer:

```powershell
& "C:\Program Files\VirtViewer v11.0-256\bin\remote-viewer.exe" "spice://127.0.0.1:5901"
```

---

## 7. Problem: UEFI zamiast instalatora

Sprawdzenie ISO:

```bash
virsh domblklist windows-server-2025
```

ISO bylo podlaczone jako `sdb`.

Sprawdzenie boot order:

```bash
virsh dumpxml windows-server-2025 | grep -A8 -B2 "<boot"
```

Bylo:

```xml
<boot dev='hd'/>
```

Zmiana:

```bash
virsh edit windows-server-2025
```

Na:

```xml
<boot dev='cdrom'/>
<boot dev='hd'/>
```

Restart:

```bash
virsh destroy windows-server-2025
virsh start windows-server-2025
```

---

## 8. Instalacja Windows Server

Wybrano:

```text
Windows Server 2025 Standard Evaluation (Desktop Experience)
```

Desktop Experience zawiera GUI. Konto `Administrator` otrzymalo haslo, ktore nie jest przechowywane w repo.

---

## 9. Hostname

```powershell
Rename-Computer -NewName "SRV-DC01" -Restart
```

---

## 10. Rozpoznanie sieci

```powershell
Get-NetIPConfiguration
Get-NetAdapter
```

Potwierdzono:
- interfejs `Ethernet`,
- ifIndex `5`,
- MAC `52-54-00-BF-04-63`,
- poczatkowo DHCP,
- adres `192.168.122.24`,
- brama i DNS `192.168.122.1`.

---

## 11. Rezerwacja IP w libvirt

Zakres DHCP:

```text
192.168.122.2 - 192.168.122.254
```

MAC VM:

```text
52:54:00:bf:04:63
```

Rezerwacja:

```bash
virsh net-update default add ip-dhcp-host \
  "<host mac='52:54:00:bf:04:63' ip='192.168.122.24'/>" \
  --live --config
```

Rezerwacja ogranicza ryzyko konfliktu z DHCP i gwarantuje ten sam IP, gdyby DHCP zostalo ponownie wlaczone.

---

## 12. Statyczny IPv4

```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Disabled
```

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" `
  -IPAddress 192.168.122.24 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.122.1
```

### Rzeczywiste bledy

Literowki:

```text
-IPAdress
-PrefixLenght
```

powodowaly `NamedParameterNotFound`.

---

## 13. DNS

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.122.1
```

Na Dniu 24 DNS zostanie dostosowany do roli AD DS/DNS.

---

## 14. Testy sieci

```powershell
ping 192.168.122.1
ping 1.1.1.1
nslookup example.com
```

Rzeczywiste wyniki:
- brama: 4/4, 0% strat;
- Internet po IP: 4/4, 0% strat;
- DNS: poprawna odpowiedz dla `example.com`.

---

## 15. RDP

W Server Manager -> Local Server:
- `Remote Desktop: Disabled`
- wybrano `Allow remote connections to this computer`;
- Windows wlaczyl odpowiedni wyjatek zapory.

Weryfikacja serwera:

```powershell
Get-NetTCPConnection -LocalPort 3389 -State Listen
```

---

## 16. RDP z fizycznego Windows - troubleshooting

Pierwszy test:

```powershell
Test-NetConnection 192.168.122.24 -Port 3389
```

Wynik poczatkowy: `False`.

Trasa:

```powershell
route print | findstr 192.168.122
```

Potwierdzono `192.168.122.0/24` przez `192.168.0.51`.

Ubuntu:

```bash
sudo iptables -L LIBVIRT_FWI -n -v --line-numbers
```

Libvirt odrzucal nowe polaczenia do `virbr0`.

Dodano waski wyjatek:

```bash
sudo iptables -I LIBVIRT_FWI 2 \
  -i eno1 -o virbr0 \
  -s 192.168.0.99 -d 192.168.122.24 \
  -p tcp --dport 3389 -j ACCEPT
```

Ponowny test:

```text
TcpTestSucceeded : True
```

> Ta reczna regula iptables jest traktowana jako tymczasowa i moze zniknac po restarcie/przebudowie regul libvirt.

Polaczenie:

```powershell
mstsc
```

Komputer:

```text
192.168.122.24
```

Uzytkownik:

```text
Administrator
```

W labie po polaczeniu pojawil sie czarny ekran. Zamkniecie rownolegle otwartej konsoli SPICE/Remote Viewer przywrocilo poprawny obraz RDP.

---

## 17. Rollback

Usuniecie DHCP reservation:

```bash
virsh net-update default delete ip-dhcp-host \
  "<host mac='52:54:00:bf:04:63' ip='192.168.122.24'/>" \
  --live --config
```

Usuniecie tymczasowego wyjatku RDP:

```bash
sudo iptables -D LIBVIRT_FWI \
  -i eno1 -o virbr0 \
  -s 192.168.0.99 -d 192.168.122.24 \
  -p tcp --dport 3389 -j ACCEPT
```

Powrot Windows do DHCP:

```powershell
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses
```

---

## 18. Do sprawdzenia

Na screenie `Local Server` strefa czasowa byla ustawiona na `Pacific Time`. W tej sesji nie zweryfikowano jej zmiany, wiec nie oznaczamy tego jako wykonane.

---

## Nastepny krok

Dzien 24:
- AD DS,
- promocja `SRV-DC01` do kontrolera domeny,
- DNS domenowy,
- uzytkownicy i grupy,
- dolaczenie klienta do domeny.
