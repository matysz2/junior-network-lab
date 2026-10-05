# Dzien 24 - Active Directory - postep

## Status
Dzien 24 wykonany praktycznie w zakresie AD DS, kontrolera domeny, DNS domenowego, OU, uzytkownika i grupy.
Domain Join klienta zostal omowiony i udokumentowany, ale nie wykonany praktycznie, poniewaz aktualny klient zwrocil Windows 10 Home / Core.

## Wykonane
- Snapshot VM `pre-AD-DS`.
- Instalacja roli Active Directory Domain Services.
- Promocja `SRV-DC01` do pierwszego DC.
- Nowy forest i domena `corp.example.com`.
- NetBIOS `CORP`.
- Forest/Domain Functional Level: Windows Server 2025.
- DNS + Global Catalog.
- DSRM password ustawione, nie zapisane w dokumentacji.
- `Get-ADDomain` potwierdzil domenę.
- DNS klienta na DC po promocji: `127.0.0.1`.
- Strefy `corp.example.com` i `_msdcs.corp.example.com`, AD-integrated.
- `Resolve-DnsName SRV-DC01.corp.example.com` -> `192.168.122.24`.
- SRV `_ldap._tcp.dc._msdcs.corp.example.com` -> `SRV-DC01`, port 389.
- OU `Firma` oraz `Uzytkownicy`, `Grupy`, `Komputery`.
- Uzytkownik `Jan Kowalski`, `jan.kowalski`, wymuszona zmiana hasla przy pierwszym logowaniu.
- Grupa `GG_Pracownicy`, Global/Security.
- Jan dodany do `GG_Pracownicy`.
- `Get-ADGroupMember -Identity "GG_Pracownicy"` potwierdzil czlonkostwo.

## Troubleshooting
- Po restarcie VM RDP bylo blokowane przez kolejnosc regul `LIBVIRT_FWI`; usluga RDP na DC byla osiagalna bezposrednio z Ubuntu.
- Zolty warning DNS delegation nie blokowal promocji; prerequisites przeszly.
- Przy ponownym tworzeniu `GG_Pracownicy` pojawil sie poprawny komunikat, ze grupa juz istnieje.
- Obecny klient: `Windows 10 Home`, `WindowsEditionId=Core`; nie obsluguje klasycznego Domain Join.

## Do wykonania pozniej
Po uruchomieniu Windows 10/11 Pro:
1. Routing do `192.168.122.0/24` przez Ubuntu `192.168.0.51`.
2. Dostep klient -> DC przez firewall labu.
3. DNS klienta `192.168.122.24`.
4. Test rekordow A/SRV i portow AD.
5. Join do `corp.example.com`.
6. Restart i logowanie `CORP\jan.kowalski`.
7. Weryfikacja obiektu komputera i przeniesienie do `OU=Komputery,OU=Firma,...`.

## Nastepny krok
Dzien 25: DHCP, podstawy GPO, SMB, uprawnienia Share + NTFS, backup i odtwarzanie malego przykladu.
