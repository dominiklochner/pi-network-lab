# Phase 4 — Firewall (ufw)

*(Network+ Domain 4: Network Security)*

## Konzept: Verteidigung in Tiefe + Least Privilege

Die FritzBox schützt nur vor dem Internet — innerhalb des Heimnetzes war der Pi bis hierhin für jedes Gerät komplett offen (SSH, Pi-hole-Weboberfläche, DNS, WireGuard, alles uneingeschränkt erreichbar). Zwei Prinzipien angewendet:

- **Defense in Depth:** ein zusätzlicher Schutzlayer direkt auf dem Pi, unabhängig vom Router — falls z.B. ein anderes Gerät im Heimnetz kompromittiert wird
- **Least Privilege:** Grundhaltung umgedreht von "alles erlauben außer verboten" zu "alles verbieten außer explizit erlaubt"

## Grundregeln

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Eingehend grundsätzlich blockiert, ausgehend weiterhin erlaubt (der Pi braucht selbst Zugriff nach draußen — Upstream-DNS, `apt update`). Bewusst in dieser Reihenfolge gesetzt, **bevor** die Firewall aktiviert wird — kein Risiko, sich selbst auszusperren.

## Gezielt freigegebene Ports

```bash
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # Pi-hole Weboberfläche
sudo ufw allow 53          # DNS (TCP + UDP zugleich, ohne Protokoll-Suffix)
sudo ufw allow 51820/udp   # WireGuard
```

Bewusst **keine** Quell-IP-Einschränkung bei SSH: eine Beschränkung auf das Heimnetz-Subnetz hätte den VPN-Zugriff auf SSH gebrochen, weil eingehende Pakete über den Tunnel mit der VPN-Adresse (`10.10.10.2`) ankommen, nicht mit einer Heimnetz-Adresse. Der Router selbst lässt SSH von außen ohnehin nicht durch (keine Portfreigabe dafür).

## Stolperstein: `deny (routed)` als Default

`sudo ufw enable` zeigte in der Statusausgabe `Default: deny (incoming), allow (outgoing), deny (routed)` — der dritte Wert war neu und kritisch: "routed" betrifft Traffic, der **durch** den Pi hindurchgeleitet wird, genau das was der WireGuard-Tunnel für den Zugriff aufs restliche Heimnetz braucht (Phase 3). Ohne Gegenmaßnahme hätte ufw die eigene `PostUp`-Weiterleitungsregel aus der WireGuard-Config potenziell überschrieben.

Lösung — der offiziell vorgesehene ufw-Weg für genau diesen Fall:

```bash
sudo ufw route allow in on wg0 out on eth0
```

Erlaubt gezielt: Traffic, der über die WireGuard-Schnittstelle reinkommt und über die Kabel-Schnittstelle ins Heimnetz weiter soll.

## Verifikation

Kompletter Regressionstest von Phase 3 wiederholt (WLAN aus, Mobilfunk-Hotspot, Tunnel aktiv, `ping 192.168.178.1`) — erst ein Fehlversuch (Tunnel vermutlich noch nicht verbunden), dann erfolgreich mit normaler Round-Trip-Zeit (35–72 ms). Bestätigt: Firewall aktiv, VPN-Weiterleitung weiterhin intakt.

## Status: abgeschlossen ✅

## Nächster Schritt

→ Phase 5: Troubleshooting bewusst üben — `docs/05-troubleshooting.md` (folgt)
