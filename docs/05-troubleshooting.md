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

