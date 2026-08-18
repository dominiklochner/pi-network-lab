# Phase 5 — Troubleshooting bewusst üben

*(Network+ Domain 5 — mit 24% der größte Prüfungsbereich)*

Für jede Übung: Fehler bewusst einbauen, dann nach der offiziellen 7-Schritte-Methodik lösen:

1. Problem identifizieren
2. Theorie zur wahrscheinlichen Ursache aufstellen
3. Theorie testen
4. Lösungsplan erstellen
5. Lösung umsetzen
6. Vollständige Funktion verifizieren
7. Dokumentieren

---

## Trouble-Ticket #1: Falscher Standard-Gateway

**Symptom:** Lokales Netz funktioniert normal (DNS-Anfragen an `192.168.178.1` erfolgreich), aber jede Verbindung zu einem Ziel außerhalb des lokalen Subnetzes (z. B. `ping 8.8.8.8`) schlägt sofort fehl mit `Destination Host Unreachable`, gemeldet vom Pi selbst (`192.168.178.47`) — das Paket verließ den Pi nie.

**Erste Theorie (falsch):** DNS-Server ist das Problem. Verworfen bei Schritt 3, weil `ping 8.8.8.8` eine reine IP-Adresse anspricht und dafür gar keine DNS-Auflösung nötig ist — ein Test, der DNS gar nicht benutzt, kann nicht durch DNS verursacht sein.

**Korrigierte Theorie:** Standard-Gateway ist falsch — die einzige Einstellung, die bestimmt, wie ein Gerät Ziele außerhalb seines eigenen Subnetzes erreicht.

**Test:** `ip route` zeigte `default via 192.168.178.99` — eine nicht existierende Adresse statt der echten FritzBox (`192.168.178.1`).

**Lösung:**
```bash
sudo nmcli connection modify netplan-eth0 ipv4.gateway 192.168.178.1
sudo nmcli connection down netplan-eth0 && sudo nmcli connection up netplan-eth0
```

**Verifiziert:** `ping 8.8.8.8` wieder erfolgreich.

**Erkenntnis:** DNS-Erfolg beweist nicht automatisch generelle Internet-Erreichbarkeit. Eine DNS-Anfrage an einen lokalen Resolver (hier: die FritzBox) bleibt oft komplett im lokalen Netz — der Resolver selbst erledigt die eigentliche Anfrage nach draußen, das eigene Routing des Client-Geräts wird dabei nie geprüft. Ein echter Konnektivitätstest braucht ein Ziel, das das eigene Gerät zwingt, selbst zu routen (z. B. Ping auf eine rohe, garantiert externe IP).

---

## Trouble-Ticket #2: Falscher DNS-Upstream in Pi-hole

**Symptom:** Anfragen über den normalen System-Resolver (FritzBox, `192.168.178.1`) liefen weiterhin fehlerfrei — täuschte zunächst darüber hinweg, dass überhaupt ein Problem vorliegt. Erst eine gezielte Anfrage direkt an Pi-hole (`dig <neue-domain> @127.0.0.1`) für eine noch nicht gecachte Domain zeigte den Fehler: `communications error to 127.0.0.1#53: timed out`.

**Wichtiger Zwischenschritt:** die ersten Tests (`ping github.com`, `ping wikipedia.com`, `nslookup booking.com`, …) liefen alle über die FritzBox als Standard-Resolver (sichtbar an `Server: 192.168.178.1` in der `nslookup`-Ausgabe) — der Pi selbst nutzt aus einer bewussten Entscheidung in Phase 1/2 weiterhin die FritzBox als System-DNS, Pi-hole wurde nie zum echten Standard-Resolver gemacht (nur lokal getestet, nicht ausgerollt). Diese Tests konnten den Fehler also grundsätzlich nicht aufdecken, da sie Pi-hole nie erreichten.

**Theorie:** Pi-hole selbst kann Anfragen, die es nicht aus eigenem Cache/eigenen Sperrlisten beantworten kann, nicht mehr an einen funktionierenden Upstream-Server weiterleiten — konkret der in Pi-hole unter Settings → DNS konfigurierte Upstream, **nicht** die System-DNS-Einstellung des Pi selbst.

**Test:** In Pi-hole Settings → DNS stand tatsächlich `192.0.2.1` (eine reservierte, nie erreichbare Test-Adresse) als Upstream, statt des eigentlich konfigurierten Quad9 (`9.9.9.9`).

**Lösung:** Quad9 (Secured, `9.9.9.9`) in den Pi-hole-DNS-Einstellungen wieder aktiviert, fehlerhaften Custom-Eintrag entfernt.

**Verifiziert:** `dig aboutyou.com @127.0.0.1` liefert wieder eine echte Antwort (11 ms Antwortzeit).

**Erkenntnis:** Ein Dienst kann "teilweise funktionieren" — Pi-hole selbst lief die ganze Zeit, beantwortete gecachte/geblockte Domains weiterhin sofort, nur die Weiterleitung an Upstream war betroffen. Zeigt nochmal denselben Grundsatz wie Ticket #1: genau das testen, was tatsächlich den vermuteten fehlerhaften Pfad durchläuft — nicht das, was zufällig danebenliegt und deshalb weiterhin funktioniert.

---
