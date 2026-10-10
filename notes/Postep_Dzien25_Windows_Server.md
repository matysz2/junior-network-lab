# Postęp - Dzień 25 - Windows Server

Data: 2026-10-09

## Wykonane i zweryfikowane
- DHCP Server zainstalowany i uruchomiony na SRV-DC01.
- Serwer DHCP autoryzowany w AD.
- Drugi interfejs lab: 192.168.10.1/24 w vlan10-lab.
- Zakres DHCP: 192.168.10.100-192.168.10.200.
- DNS dla zakresu: 192.168.10.1; domena: corp.example.com.
- Binding DHCP wyłączony na interfejsie 192.168.122.24 i pozostawiony na Ethernet 2.
- Test DORA zakończony sukcesem; klient dostał 192.168.10.100.
- Utworzono GPO LAB-GPO-Dzien25 i podlinkowano do OU Domain Controllers.
- GPO utworzyło wpis HKLM\SOFTWARE\JuniorNetworkLab\Dzien25_GPO = GPO_DZIALA.
- `gpupdate /target:computer /force` zakończył się sukcesem.
- Utworzono udział SMB `\\SRV-DC01\Dzien25`.
- Utworzono grupę AD `GG_Dzien25_RW`.
- Share Permissions: GG_Dzien25_RW = Change.
- NTFS: GG_Dzien25_RW = Modify (OI)(CI).
- Zapisano kopię ACL do `C:\LabShares\Dzien25_ACL_backup.txt`.
- Wyłączono dziedziczenie ACL na folderze lab i usunięto BUILTIN\Users.
- Dodano jan.kowalski do GG_Dzien25_RW.
- Zdiagnozowano błąd SMB: konto wymagało zmiany hasła / PasswordExpired=True / pwdLastSet=0.
- Po resecie hasła użytkownik odczytał udział i utworzył `jan_test.txt`.
- Wykonano prosty backup `test.txt` do `C:\LabBackup\Dzien25`.
- Zweryfikowano SHA256, usunięto plik źródłowy, odtworzono go i ponownie potwierdzono identyczny SHA256.

## Trudności / rzeczy do powtórki
- Brak dostępu SMB nie musi oznaczać złych ACL: osobno sprawdzamy port 445, istnienie udziału, uwierzytelnianie/hasło i uprawnienia.
- Share Permissions i NTFS Permissions to dwa różne poziomy kontroli.
- `pwdLastSet = 0` wskazywało na wymaganie zmiany hasła przy następnym logowaniu.
- Backup w innym katalogu na tym samym serwerze był tylko ćwiczeniem, nie pełną strategią backupową.

## Quiz
Wynik: 6/7 (ok. 86%).
Błąd: przyczyna problemu SMB została początkowo wskazana jako NTFS, podczas gdy faktycznym problemem było hasło użytkownika wymagające zmiany.

## Następny krok
Dzień 26: NTP, syslog, podstawy SNMP i monitoringu oraz prosty skrypt diagnostyczny Bash lub PowerShell.
