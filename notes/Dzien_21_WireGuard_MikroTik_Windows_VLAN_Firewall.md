# Dzień 21 - WireGuard na MikroTik

## Cel
Uruchomienie WireGuard na MikroTik CHR, zestawienie klienta Windows, dostęp przez VPN do VLAN10 i ograniczenie dostępu do VLAN20 firewallem.

## Topologia
```text
Windows 192.168.0.99 / VPN 10.70.0.2
        |
        | WireGuard UDP 13231
        v
MikroTik CHR 192.168.122.75 / wg-remote 10.70.0.1
        |
        +-- VLAN10 192.168.10.0/24 -> klient10 192.168.10.200 (ALLOW)
        +-- VLAN20 192.168.20.0/24 -> klient20 192.168.20.200 (DROP)
```

## RouterOS
```routeros
/interface wireguard add name=wg-remote listen-port=13231
/ip address add address=10.70.0.1/24 interface=wg-remote
/interface wireguard peers add interface=wg-remote public-key="<PUBLIC_KEY_WINDOWS>" allowed-address=10.70.0.2/32 persistent-keepalive=25
```

## Windows
```ini
[Interface]
PrivateKey = <PRIVATE_KEY_WINDOWS>
Address = 10.70.0.2/24

[Peer]
PublicKey = <PUBLIC_KEY_MIKROTIKA>
AllowedIPs = 10.70.0.1/32
Endpoint = 192.168.122.75:13231
PersistentKeepalive = 25
```

Prywatne klucze nie są publikowane.

## Diagnostyka libvirt
```bash
sudo iptables -L FORWARD -n -v --line-numbers
sudo iptables -L LIBVIRT_FWI -n -v --line-numbers
sudo iptables -I LIBVIRT_FWI 3 -i eno1 -o virbr0 -s 192.168.0.99 -d 192.168.122.75 -p udp --dport 13231 -j ACCEPT
sudo tcpdump -ni virbr0 udp port 13231 -c 4
```

Tcpdump potwierdził ruch w obie strony.

## Handshake
`/interface wireguard peers print detail` pokazało:
- bieżący endpoint Windowsa,
- `rx > 0`,
- `tx > 0`,
- świeży `last-handshake`.

Windows -> `10.70.0.1`: 4/4, 0% strat.

## VLAN10
Odtworzono `klient10 = 192.168.10.200/24`.

Windows:
```ini
AllowedIPs = 10.70.0.1/32, 192.168.10.0/24
```

Windows -> `192.168.10.200`: 4/4, 0% strat.

## AllowedIPs vs routing vs firewall
```text
AllowedIPs = co klient wysyła przez VPN
Routing    = którędy MikroTik wysyła pakiet dalej
Firewall   = czy ruch jest dozwolony
```

- jeden host: `192.168.10.50/32`
- cała podsieć: `192.168.10.0/24`

## Firewall
```routeros
/ip firewall filter add chain=forward src-address=10.70.0.2/32 dst-address=192.168.10.0/24 action=accept comment="ALLOW VPN to VLAN10"
/ip firewall filter add chain=forward src-address=10.70.0.2/32 action=drop comment="BLOCK VPN to other networks"
```

## VLAN20
Odtworzono `klient20 = 192.168.20.200/24`.

Na czas testu:
```ini
AllowedIPs = 10.70.0.1/32, 192.168.10.0/24, 192.168.20.0/24
```

Windows -> `192.168.20.200`: timeout.

`BLOCK VPN to other networks`: 4 pakiety / 240 B.

## INPUT vs FORWARD
```text
VPN -> 192.168.20.1   = input
VPN -> 192.168.20.200 = forward
```

## Rollback
```routeros
/ip firewall filter remove [find comment="ALLOW VPN to VLAN10"]
/ip firewall filter remove [find comment="BLOCK VPN to other networks"]
```

```bash
sudo iptables -D LIBVIRT_FWI -i eno1 -o virbr0 -s 192.168.0.99 -d 192.168.122.75 -p udp --dport 13231 -j ACCEPT
```

## Wyniki
- Windows -> 10.70.0.1: 4/4, 0% strat
- Windows -> 192.168.10.200: 4/4, 0% strat
- Windows -> 192.168.20.200: timeout
- DROP: 4 pakiety / 240 B

## Następny dzień
Dzień 22: CGNAT, split/full tunnel, DNS, MTU, Tailscale.
