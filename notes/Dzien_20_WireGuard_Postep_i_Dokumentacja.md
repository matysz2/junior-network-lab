# Dzień 20 - WireGuard - stan końcowy

## Wykonane praktycznie

- zainstalowano `wireguard-tools`;
- utworzono i przetestowano ręczny lab WireGuard;
- potwierdzono handshake i ping w obie strony w labie;
- przygotowano i uruchomiono `wg-prod` na Debianie;
- serwer Debian: `192.168.0.51`, VPN `10.60.0.1/24`, UDP `51821`;
- zainstalowano WireGuard GUI na Windows;
- utworzono rzeczywisty klient Windows `10.60.0.3/24`;
- Endpoint klienta: `192.168.0.51:51821`;
- AllowedIPs klienta: `10.60.0.1/32`;
- PersistentKeepalive: `25`;
- Debian zobaczył świeży `latest handshake` z Windows;
- ping Windows -> `10.60.0.1`: 4/4, 0% strat, średnio 4 ms;
- po przypadkowym ujawnieniu klucza prywatnego wykonano rotację klucza.

## Wniosek

Praktyczny scenariusz Windows GUI -> Debian WireGuard został wykonany i zweryfikowany.

## Do powtórki

- różnica między `Endpoint` i `AllowedIPs`;
- różnica między trasą systemową i AllowedIPs;
- split tunnel vs full tunnel;
- NAT/CGNAT;
- `interface UP` nie oznacza automatycznie działającego handshake.

## Następny krok

Dzień 21: WireGuard na MikroTik - peer, klucze, endpoint, allowed-address, trasy, firewall, keepalive i powtórka tygodniowa.
