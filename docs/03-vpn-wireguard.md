# Phase 3 — VPN (WireGuard)

*(Network+ Domain 2: Network Implementation)*

## Konzept

Bisher nur im Heimnetz selbst erreichbar. WireGuard baut einen verschlüsselten Tunnel durchs Internet zum Pi auf — von unterwegs verhält sich das Endgerät so, als wäre es direkt im Heimnetz.

```
Mac/Handy (unterwegs) ──verschlüsselter Tunnel (Internet)──> Pi (WireGuard-Server, Port 51820/UDP) ──> Heimnetz
```

**Sicherheitsprinzip:** nur der WireGuard-Port ist von außen erreichbar, SSH & Co. bleiben komplett unsichtbar nach außen — Zugriff nur *durch* den Tunnel, nie direkt.

## Schlüsselpaare (asymmetrisch, wie SSH-Keys)

Jeder Teilnehmer ("Peer") hat ein eigenes Schlüsselpaar — Vorteil: Zugriff für ein Gerät entziehen heißt nur dessen Eintrag löschen, nicht den ganzen Tunnel neu aufsetzen.

- **Pi (Server):** `wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey`
- **Mac (Client):** über die offizielle WireGuard-App (Mac App Store) erzeugt, "Leeren Tunnel erstellen"

Private Keys verlassen nie das jeweilige Gerät — wurden zu keinem Zeitpunkt geteilt, nur die öffentlichen Schlüssel.

## Server-Konfiguration (`/etc/wireguard/wg0.conf`)

```ini
[Interface]
PrivateKey = <Pi-eigener privater Schlüssel>
Address = 10.10.10.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = mQ68bfc+cptQjM+G+BU1LOEfUEtBa+/cLjyhXq84KTk=
AllowedIPs = 10.10.10.2/32
```

Neues, eigenes VPN-Subnetz (`10.10.10.0/24`), getrennt vom Heimnetz (`192.168.178.0/24`), Pi = `.1`, Mac = `.2`. `PostUp`/`PostDown` aktivieren automatisch IP-Weiterleitung + NAT-Masquerading beim Start/Stopp des Tunnels, damit Tunnel-Traffic auch ins restliche Heimnetz kommt (nicht nur bis zum Pi).

Vorbereitend: `net.ipv4.ip_forward=1` über `/etc/sysctl.d/99-wireguard.conf` dauerhaft aktiviert (klassische `/etc/sysctl.conf` existierte auf diesem System nicht mehr — moderne Distributionen nutzen stattdessen Drop-in-Dateien unter `/etc/sysctl.d/`).

Dienst: `sudo systemctl enable --now wg-quick@wg0`

## Client-Konfiguration (Mac, WireGuard-App)

```ini
[Interface]
PrivateKey = <Mac-eigener privater Schlüssel, automatisch von der App gesetzt>
Address = 10.10.10.2/32

[Peer]
PublicKey = Cqth9Skp8hG7/J2ushI/bXkrd+PedzyzZ7+NcJfD9y8=
Endpoint = mk29ybuf96i4i2tj.myfritz.net:51820
AllowedIPs = 192.168.178.0/24, 10.10.10.0/24
PersistentKeepalive = 25
```

Bewusst **Split Tunnel** (nur Heimnetz + VPN-Subnetz durch den Tunnel), kein `0.0.0.0/0` (Full Tunnel würde den kompletten Internet-Traffic umleiten — nicht das Ziel hier).

## Netzwerk-seitige Voraussetzungen

- **Öffentliche IPv4 verifiziert** (kein DS-Lite/CGNAT): `curl -4 ifconfig.me` auf dem Pi mit der in der FritzBox angezeigten WAN-IP verglichen — identisch, also eine echte eigene IPv4, keine geteilte Provider-Adresse
- **Portfreigabe** in der FritzBox: UDP 51820 → `192.168.178.47`
- **DynDNS** über MyFRITZ!: `mk29ybuf96i4i2tj.myfritz.net` — nötig, weil deutsche Privatanschlüsse i.d.R. eine dynamische (wenn auch echte) öffentliche IP haben

## Lesson Learned: Lokales Testen ist trügerisch

Erster Test (`ping 192.168.178.1` vom Mac, während im gleichen Heimnetz-WLAN) kam durch — bewies aber **nichts**: der Mac hatte für dieses Zielnetz zwei mögliche Routen (direkt übers WLAN vs. durch den Tunnel), das Betriebssystem nimmt bei gleichwertigen Zielen meist den direkten Weg. Verifiziert wurde das über `sudo wg show` auf dem Pi (Transfer-Zähler prüfen, ob sich beim Ping wirklich was bewegt) — der eigentlich abschließende Beweis kam erst durch den Test von außerhalb.

## Finaler Test (von außerhalb, WLAN aus, über Mobilfunk-Hotspot)

```bash
ping 192.168.178.1
```

Erfolgreich — diesmal eindeutig, da ohne Tunnel technisch gar kein Weg ins Heimnetz existiert hätte. Zusätzlich Pi-hole-Weboberfläche (`http://192.168.178.47/admin`) von unterwegs erfolgreich geladen — voller Zugriff aufs Heimnetz von außen bestätigt.

## Status: abgeschlossen ✅

## Nächster Schritt

→ Phase 4: Firewall (ufw) — `docs/04-firewall.md` (folgt)
