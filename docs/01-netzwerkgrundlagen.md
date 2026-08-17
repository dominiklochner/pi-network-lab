# Phase 1 — Netzwerkgrundlagen: feste IP

*(Network+ Domain 1: Networking Concepts)*

## Ausgangslage

Der Pi bezog seine IP per DHCP und war gleichzeitig über zwei Interfaces aktiv: `eth0` (Kabel, `192.168.178.47`) und `wlan0` (WLAN, `192.168.178.48`), jeweils mit eigener Default-Route (Kabel priorisiert über Metric). Für einen Server, der dauerhaft unter einer festen, vorhersagbaren Adresse erreichbar sein soll, ist das unnötige Komplexität — Ziel: eine feste IP, nur über Kabel.

## Wer verwaltet das Netzwerk wirklich?

Erster Blick auf `nmcli connection show` zeigte Verbindungsnamen wie `netplan-eth0` — sah zunächst nach netplan als Config-Autorität aus. Verifiziert mit `sudo cat /etc/netplan/*.yaml`: `renderer: NetworkManager` + `passthrough`-Block bestätigen das Gegenteil — **NetworkManager ist die eigentliche Quelle der Wahrheit**, netplan schreibt hier nur eine Spiegel-Datei zur Kompatibilität. Direktes Ändern per `nmcli` ist also korrekt, kein Risiko dass netplan die Änderung überschreibt.

## Aufräumen vor der Config-Änderung

Verwaiste Docker-Netzwerk-Interfaces (`docker0`, zwei `br-*`-Bridges) waren trotz Docker-Deinstallation noch als Kernel-Interfaces vorhanden, samt zugehöriger "extern verbundener" NetworkManager-Profile:

```bash
sudo ip link delete docker0
sudo ip link delete br-9f69e6b7656c
sudo ip link delete br-bee7312e6da6
```

Die dazugehörigen NetworkManager-Profile verschwanden danach automatisch (waren nur an die Existenz der Interfaces gekoppelt).

## Feste IP setzen

```bash
sudo nmcli connection modify netplan-eth0 \
  ipv4.addresses 192.168.178.47/24 \
  ipv4.gateway 192.168.178.1 \
  ipv4.dns "192.168.178.1" \
  ipv4.method manual

sudo nmcli connection down netplan-eth0 && sudo nmcli connection up netplan-eth0
```

Gleiche IP wie vorher per DHCP übernommen (`192.168.178.47`) — keine Umgewöhnung nötig. DNS zeigt vorerst auf die FritzBox, wird in Phase 2 auf den Pi selbst (`127.0.0.1`, Pi-hole) umgestellt.

WLAN deaktiviert statt gelöscht (bleibt als Reserve-Zugang gespeichert, verbindet sich nur nicht mehr automatisch):

```bash
sudo nmcli connection down netplan-wlan0-UnserWlan
sudo nmcli connection modify netplan-wlan0-UnserWlan connection.autoconnect no
```

## Stolperstein & Lesson Learned (praktisches Troubleshooting-Beispiel)

Direkt nach dem Deaktivieren von WLAN brach die aktive SSH-Sitzung ab. Grund: die Sitzung lief tatsächlich noch über `wlan0`, nicht über das vermeintlich genutzte Kabel — trotz der Annahme, per Ethernet verbunden zu sein. Verifiziert durch erneutes Einloggen **explizit über die feste IP** (`ssh pi@192.168.178.47`, bewusst nicht über `raspberrypi.local`, da mDNS-Namen auf eine beliebige der verfügbaren Schnittstellen zeigen können) — Verbindung stand sofort, bestätigt dass das Kabel tatsächlich aktiv ist und die feste IP funktioniert.

**Erkenntnis:** Bei mehreren aktiven Interfaces nie annehmen, über welches man gerade verbunden ist — explizit über die Ziel-IP prüfen, nicht über einen Hostnamen, der mehrdeutig auflösen kann.

## Offene Baustelle (nicht dringend)

`192.168.178.47` liegt im DHCP-Adresspool der FritzBox — theoretisches Risiko einer späteren Adress-Kollision. Optional für später: DHCP-Pool in der FritzBox verkleinern oder eine MAC-basierte Reservierung eintragen.

## Status: abgeschlossen ✅

## Nächster Schritt

→ Phase 2: Pi-hole (DNS) — `docs/02-dns-pihole.md` (folgt)
