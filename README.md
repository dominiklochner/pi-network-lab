# Pi Network Lab

Ein Raspberry Pi als zentraler Netzwerk-Service-Server im Heimnetz — Schritt für Schritt aufgebaut und dokumentiert entlang der CompTIA-Network+-Prüfungsdomänen. Praxisprojekt fürs Portfolio, gleichzeitig direkte Vorbereitung auf die Zertifizierung.

## Ziel

Kernnetzwerkdienste (DNS, VPN, Firewall) produktiv auf einem Raspberry Pi betreiben — jede Phase einer Network+-Domäne zugeordnet, inklusive bewusst eingebauter Fehler zum Üben der offiziellen Troubleshooting-Methodik.

## Phasen

| Phase | Thema | Network+ Domäne |
|---|---|---|
| 0 | Setup & Bestandsaufnahme | — |
| 1 | Netzwerkgrundlagen (statische IP, Linux-Networking) | Domain 1: Networking Concepts |
| 2 | DNS-Server (Pi-hole) | Domain 1 & 3 |
| 3 | VPN (WireGuard) | Domain 2: Network Implementation |
| 4 | Firewall (ufw) | Domain 4: Network Security |
| 5 | Troubleshooting (bewusst kaputt machen & lösen) | Domain 5: Network Troubleshooting |

## Umgebung

- Raspberry Pi (headless, Raspberry Pi OS Lite)
- MacBook Pro (M1) als Steuerzentrale / SSH-Client
- Kein zusätzlicher Server nötig

## Struktur

```
docs/         Ein Dokument pro Phase
diagrams/     Netzwerkdiagramme (Mermaid)
screenshots/  Belege pro Phase
```

## Verwandtes Projekt

Ergänzend zu [homelab-network-project](https://github.com/dominiklochner/homelab-network-project) (Windows Server AD, OPNsense, VLANs) — dieses Projekt deckt die Linux-/Netzwerkdienste-Seite ab.
