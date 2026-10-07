# Junior Network Lab

Projekt laboratoryjny przygotowujący do pracy na stanowisku
Junior Network Engineer.

## Cel projektu

Budowa i dokumentacja sieci małego biura LAB-BIURO.

Projekt obejmuje m.in.:

- adresację IPv4 i IPv6
- VLAN i trunk 802.1Q
- routing między VLAN-ami
- STP / RSTP
- EtherChannel / LACP
- OSPF
- DNS i DHCP
- Debian/Linux
- MikroTik RouterOS
- NAT i firewall
- WireGuard VPN
- Windows Server
- Active Directory
- Group Policy
- udziały SMB i NTFS
- backup i odtwarzanie
- diagnostykę sieci

## Środowisko laboratoryjne

### Linux
- Ubuntu Server 24.04 LTS
- adres LAN: `192.168.0.51/24`
- SSH
- WireGuard

### Cisco
Laboratoria wykonane w Cisco Packet Tracer:

- VLAN 10 – PRACOWNICY
- VLAN 20 – GOSCIE
- trunk 802.1Q
- Router-on-a-Stick
- STP / RSTP
- EtherChannel LACP
- OSPF
- IPv6

### MikroTik
MikroTik CHR / RouterOS:

- WinBox
- bridge
- VLAN
- DHCP
- routing
- NAT
- firewall
- WireGuard

### Windows Server

Windows Server 2025:

- host: `SRV-DC01`
- domena: `corp.example.com`
- NetBIOS: `CORP`
- Active Directory Domain Services
- DNS domenowy
- kontroler domeny

## Status projektu

| Etap | Status |
|---|---|
| Podstawy sieci i adresacja | ✅ wykonane |
| VLAN / trunk / routing | ✅ wykonane |
| STP / RSTP / EtherChannel | ✅ wykonane |
| OSPF i IPv6 | ✅ wykonane |
| Debian / diagnostyka | ✅ wykonane |
| NAT / firewall | ✅ wykonane |
| MikroTik RouterOS | ✅ wykonane |
| WireGuard Debian ↔ Windows | ✅ wykonane |
| WireGuard MikroTik | ✅ wykonane |
| Windows Server 2025 | ✅ wykonane |
| Active Directory `corp.example.com` | ✅ wykonane |
| DHCP Windows Server | 🔄 w trakcie |
| Group Policy (GPO) | ⏳ do wykonania |
| SMB + NTFS | ⏳ do wykonania |
| Backup / Restore | ⏳ do wykonania |
| Monitoring | ⏳ do wykonania |
| Projekt końcowy | ⏳ do wykonania |

## Aktualny etap – Dzień 25

Potwierdzono działanie Active Directory:

```powershell
Get-ADDomain