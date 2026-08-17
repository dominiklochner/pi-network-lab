# Phase 2 — DNS-Server (Pi-hole)

*(Network+ Domain 1 & 3: Networking Concepts / Network Operations)*

## Konzept: DNS-Sinkholing

Normalerweise übersetzt ein DNS-Server (hier bisher: die FritzBox) Domainnamen in IP-Adressen für jede angefragte Domain gleichermaßen — auch für Werbe-/Tracking-Server. Pi-hole wird selbst zum DNS-Server und vergleicht jede Anfrage gegen eine Sperrliste bekannter Werbe-/Tracking-Domains:

```
Gerät ──DNS-Anfrage──> Pi-hole ──(auf Sperrliste?)──┬─ ja  → Antwort: 0.0.0.0 (Sackgasse, Pi-hole antwortet selbst/"aa")
                                                      └─ nein → an Upstream weiterleiten → echte IP zurückgeben (kein "aa", nur durchgereicht)
```

## Entscheidungen bei der Installation

- **Interface:** `eth0` — der Installer erkannte automatisch die feste IP aus Phase 1 und empfahl sie ausdrücklich (Bestätigung, dass eine feste IP für einen DNS-Server Voraussetzung ist)
- **Upstream-DNS:** Quad9 (Secured, `9.9.9.9`, ohne ECS) statt Cloudflare — einziger Anbieter der drei Standardoptionen mit echtem Sicherheits-Mehrwert (blockt bekannte Malware-/Phishing-Domains direkt auf DNS-Ebene), gemeinnützig, kein werbefinanziertes Geschäftsmodell. Bewusst **nicht** die "Unfiltered"-Variante (9.9.9.10, hätte den Sicherheitsvorteil zunichtegemacht) oder "Secured+ECS" (9.9.9.11, tauscht Anonymität gegen CDN-Optimierung)
- **Sperrliste:** Standard (StevenBlack)
- **Protokolle:** nur IPv4
- **Weboberfläche:** ja → `http://192.168.178.47/admin`
- **Query-Logging:** ja (für die Testphase sinnvoll — sichtbar machen was passiert)
- **FTL-Privacy-Level:** 0 (Show everything) — volle Sichtbarkeit für die Lernphase, da nur lokal getestet wird und noch keine anderen Haushaltsgeräte betroffen sind. Vor einem eventuellen netzwerkweiten Rollout nochmal auf 1–2 hochstellen, um die Privatsphäre anderer Haushaltsmitglieder zu wahren.

## Bewusst noch NICHT gemacht

Der Pi ist noch nicht netzwerkweit als DNS eingetragen (weder in der FritzBox noch in der eigenen Netzwerk-Config des Pi selbst) — Entscheidung, erstmal nur lokal zu testen, bevor der ganze Haushalt betroffen ist. Rollout ist eine spätere, bewusste Entscheidung, kein technisches Blockproblem.

## Test

```bash
dig doubleclick.net @127.0.0.1   # bekannte Tracking-Domain
dig github.com @127.0.0.1        # normale Domain
```

**Ergebnis:**
- `doubleclick.net` → `0.0.0.0`, Flags enthalten `aa` (Pi-hole antwortet selbst, authoritative) ✅ Sinkholing funktioniert
- `github.com` → echte IP (`140.82.121.4`), Flags **ohne** `aa` (nur durchgereicht von Quad9) ✅ normale Auflösung funktioniert

## Status: abgeschlossen (lokaler Test) ✅

## Nächster Schritt

→ Phase 3: WireGuard VPN — `docs/03-vpn-wireguard.md` (folgt)
