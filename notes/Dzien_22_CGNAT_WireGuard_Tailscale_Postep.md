# Dzień 22 - CGNAT, WireGuard HUB, Tailscale, DNS i MTU

## Cel dnia
- zrozumieć CGNAT i rozpoznać go,
- przećwiczyć wariant z osiągalnym koncentratorem,
- rozróżnić split tunnel i full tunnel,
- diagnozować DNS i MTU,
- zobaczyć rolę Tailscale,
- zrozumieć, jak połączyć sieci/urządzenia bez publicznego IP.

## Najważniejsza idea: brak publicznego IP po obu stronach
Dwa końcowe urządzenia nie muszą mieć publicznych adresów IP, jeżeli oba inicjują połączenie wychodzące do osiągalnego koncentratora albo korzystają z Tailscale.

```text
MikroTik/Debian za CGNAT ---> publiczny HUB/VPS <--- laptop za CGNAT
```

W ręcznym WireGuardzie publicznie osiągalny musi być HUB/VPS. W Tailscale użytkownik nie musi sam wystawiać publicznego endpointu.

## CGNAT
Shared Address Space: `100.64.0.0/10`.

Przykład podejrzenia CGNAT:
```text
WAN routera: 100.100.20.5
publiczny IP widziany w Internecie: 85.x.x.x
```

## Nasz rzeczywisty test
Panel D-Link pokazał:
```text
WAN IPv4: 85.28.166.100
```

`tailscale netcheck`:
```text
IPv4: yes, 85.28.166.100:40255
```

Wniosek: w chwili testu router miał publiczny IPv4 na WAN; nie był za CGNAT.

## WireGuard HUB - Debian
```bash
umask 077
wg genkey | tee ~/wg-hub-private.key | wg pubkey > ~/wg-hub-public.key

sudo ip link add dev wg-hub type wireguard
sudo ip address add 10.80.0.1/24 dev wg-hub
sudo wg set wg-hub private-key ~/wg-hub-private.key listen-port 51830
sudo ip link set wg-hub up
sudo wg show wg-hub
```

## MikroTik jako klient HUB-a
```routeros
/interface wireguard add name=wg-hub-client listen-port=13232
/ip address add address=10.80.0.2/24 interface=wg-hub-client

/interface wireguard peers add \
 interface=wg-hub-client \
 public-key="<PUBLIC_KEY_HUB>" \
 endpoint-address=192.168.122.1 \
 endpoint-port=51830 \
 allowed-address=10.80.0.1/32 \
 persistent-keepalive=25
```

W prawdziwym wariancie `endpoint-address` byłby publicznym adresem lub DNS-em VPS/HUB-a.

## Debian - dodanie MikroTika jako peer
```bash
sudo wg set wg-hub peer "<PUBLIC_KEY_MIKROTIK>" allowed-ips 10.80.0.2/32
sudo wg show wg-hub
```

Potwierdzono handshake.

## Test
```routeros
/ping 10.80.0.1 count=4
```

Wynik:
```text
sent=4 received=4 packet-loss=0%
min-rtt=561us avg-rtt=667us max-rtt=765us
```

## Zwykły router bez WireGuard
Router może tylko zapewniać Internet, a Debian za nim zestawia tunel wychodzący do HUB-a.

```text
Internet/CGNAT -> zwykły router -> Debian/WireGuard -> LAN
```

Aby przez Debiana udostępnić całą sieć LAN, potrzebne są IP forwarding, routing/firewall i czasem NAT.

## Tailscale
Wykonano:
```bash
tailscale status
tailscale ping 100.99.239.29
tailscale netcheck
```

Rzeczywisty `tailscale ping`:
- najpierw `via DERP(fra)`,
- potem direct `via 158.180.18.82:41641`.

DERP jest relayem. Dane nadal są szyfrowane end-to-end WireGuardem.

### netcheck - rzeczywisty wynik
```text
UDP: true
IPv4: yes, 85.28.166.100:40255
IPv6: no
MappingVariesByDestIP: false
PortMapping: UPnP, NAT-PMP
Nearest DERP: Warsaw
waw: 8.7 ms
```

## Tailscale subnet router - tylko omówione, niewykonane
Przykład dla Linux:
```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
sudo tailscale set --advertise-routes=192.168.0.0/24
```

Trasę trzeba zatwierdzić w panelu Tailscale. Nie oznaczamy tego jako wykonanego w Dniu 22.

## Split tunnel
```text
AllowedIPs = 192.168.10.0/24
```
Tylko wybrana podsieć idzie przez VPN.

## Full tunnel
```text
AllowedIPs = 0.0.0.0/0
```
Cały IPv4 idzie przez VPN. Serwer musi routować ruch i zwykle wykonywać NAT.

## DNS - diagnostyka
Jeśli IP działa, ale nazwy nie:
```powershell
ping 1.1.1.1
nslookup example.com
ipconfig /all
```

## MTU - rzeczywisty test
```powershell
ping 1.1.1.1 -f -l 1472
```
4/4, 0% strat.

```powershell
ping 1.1.1.1 -f -l 1473
```
`Packet needs to be fragmented but DF set.`

Obliczenie:
```text
1472 + 28 = 1500
```

Wniosek: zwykła ścieżka Windows -> Internet ma MTU 1500. To nie był pomiar tunelu WireGuard.

## Kolejność diagnostyki VPN
1. Endpoint.
2. Dostęp UDP.
3. Firewall wejściowy.
4. PublicKey peerów.
5. Handshake/RX/TX.
6. AllowedIPs.
7. Routing.
8. Firewall forward.
9. DNS.
10. MTU.

## Status Dnia 22
**ZAKOŃCZONY.**

Następny etap: **Dzień 23 - Windows Server VM: instalacja, hostname, adresacja, DNS i RDP.**

## Źródła
- RFC 6598: https://www.rfc-editor.org/info/rfc6598/
- Tailscale Device connectivity: https://tailscale.com/docs/reference/device-connectivity
- Tailscale DERP: https://tailscale.com/docs/reference/derp-servers
- Tailscale subnet router: https://tailscale.com/docs/features/subnet-routers/how-to/setup
- Tailscale exit nodes: https://tailscale.com/docs/features/exit-nodes
- MikroTik WireGuard: https://help.mikrotik.com/docs/spaces/ROS/pages/69664792/WireGuard
