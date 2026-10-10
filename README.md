# Junior Network Lab

Projekt laboratoryjny przygotowujący do pracy na stanowisku
**Junior Network Engineer**.

## Cel projektu

Budowa, konfiguracja, diagnostyka i dokumentacja sieci małego biura
**LAB-BIURO**.

Projekt rozwijany jest etapami i obejmuje zarówno konfigurację,
jak i testy działania, diagnostykę usterek oraz dokumentację techniczną.

## Zakres projektu

- adresacja IPv4 i IPv6
- podsieci i VLSM
- VLAN i trunk 802.1Q
- routing między VLAN-ami
- STP / RSTP
- EtherChannel / LACP
- OSPF
- DNS i DHCP
- Debian / Linux
- analiza ruchu i diagnostyka
- MikroTik RouterOS
- NAT i firewall
- WireGuard VPN
- Windows Server 2025
- Active Directory
- Group Policy
- udziały SMB i uprawnienia NTFS
- backup i odtwarzanie
- NTP
- syslog / rsyslog
- SNMP i podstawy monitoringu
- Bash – prosty skrypt diagnostyczny

---

# Środowisko laboratoryjne

## Linux

Host laboratoryjny:

- Ubuntu Server 24.04 LTS
- adres LAN: `192.168.0.51/24`
- brama: `192.168.0.1`
- SSH
- KVM / libvirt
- Docker
- WireGuard
- Tailscale
- systemd-timesyncd
- rsyslog
- SNMP

Na hoście uruchamiane są maszyny wirtualne wykorzystywane
w laboratorium.

---

## Cisco Packet Tracer

Wykonane laboratoria:

- VLAN 10 – PRACOWNICY
- VLAN 20 – GOSCIE
- porty access
- trunk 802.1Q
- Router-on-a-Stick
- routing między VLAN-ami
- STP / RSTP
- EtherChannel / LACP
- Port Security
- OSPF area 0
- podstawy IPv6

Przykładowa adresacja:

```text
VLAN 10
192.168.10.0/24
brama: 192.168.10.1

VLAN 20
192.168.20.0/24
brama: 192.168.20.1
