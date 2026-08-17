# Phase 0 — Setup & Bestandsaufnahme

## Ausgangslage

Der Pi war bereits nach diversen Online-Anleitungen konfiguriert, der genaue Zustand war unklar. Hostname, Benutzer und SSH-Zugriff (Key-Auth) funktionierten bereits gut — deshalb Entscheidung **gegen** Neu-Flash der SD-Karte (unnötiger Aufwand, Karte müsste physisch raus) und **für** einen Live-Audit per SSH: bestehendes System durchleuchten, gezielt entfernen was nicht hingehört, Rest sauber weiterverwenden.

## Schritt 1: Bestandsaufnahme

```bash
apt-mark showmanual                                      # manuell installierte Pakete
systemctl list-units --type=service --state=running      # laufende Dienste
sudo ss -tulpn                                            # offene Ports
docker ps -a                                              # falls Docker installiert: laufende/gestoppte Container
```

## Schritt 2: Befund (17.08.2026)

Gegenüber einem Standard-Raspberry-Pi-OS fielen auf:

| Fund | Einordnung | Entscheidung |
|---|---|---|
| `docker-ce` + Zubehör, `containerd.io` | volle Docker-Installation, aktiv mit 3 Containern (`homepage`, `portainer`, gestoppter `pihole`) | entfernt — Projekt nutzt bewusst kein Docker |
| `nordvpn` (`nordvpnd` lief aktiv) | VPN-Client eines Drittanbieters | entfernt — eigenes WireGuard-VPN kommt in Phase 3 |
| `mkvtoolnix`, `p7zip-full`, `build-essential`, `gdb` | Video-/Archiv-Tools, Compiler-Toolchain | entfernt — für einen Netzwerk-Service-Server irrelevant |
| `network-manager` **und** `dhcpcd-base` gleichzeitig installiert | zwei parallele Netzwerkverwaltungs-Stacks — potenzielle Konfliktquelle | **bewusst nicht angefasst**, wird gezielt in Phase 1 (feste IP) geklärt |
| `rpi-connect-lite` | Raspberry Pis eigener Cloud-Fernzugriff | optionale Entfernung, eigener Zugriffsweg (SSH-Key, später VPN) macht ihn überflüssig |
| `pidash.service` (Port 8080) | eigenes, selbstgebautes Dashboard (früheres Projekt) | entfernt (Entscheidung 17.08.2026) — sauberer Neustart, Fokus komplett auf dieses Projekt |

## Schritt 3: Aufräumen

```bash
# Docker-Container stoppen & entfernen
docker rm -f homepage portainer pihole

# Docker komplett runter
sudo apt purge -y docker-ce docker-ce-cli docker-ce-rootless-extras docker-buildx-plugin docker-compose-plugin docker-model-plugin containerd.io nordvpn
sudo rm -rf /var/lib/docker

# weitere Tutorial-Reste
sudo apt purge -y mkvtoolnix p7zip-full build-essential gdb

sudo apt autoremove -y
sudo apt autoclean
```

Danach zur Verifikation nochmal `apt-mark showmanual` — sollte danach deutlich kürzer und nachvollziehbar sein.

## Nächster Schritt

→ Phase 1: feste IP-Adresse setzen, dabei `network-manager` vs. `dhcpcd-base` klären (`docs/01-netzwerkgrundlagen.md`, folgt)

## Schritt 4: Tiefenreinigung (Reste nach der Paket-Deinstallation)

Pakete entfernen reicht nicht — es blieben Reste zurück, die eine zweite Runde brauchten:

| Fund | Ursache | Aktion |
|---|---|---|
| 2 alte Kernel-Images (`rc`-Status) + 2 Config-Reste | normale Update-Historie | `apt purge` der `rc`-Pakete |
| `/var/lib/containerd` (721 MB!), `/opt/containerd` | containerd-Paket war weg, Datenverzeichnis blieb liegen | gelöscht |
| `~/docker/` (Unterordner `homepage`, `pihole`) | Bind-Mount-Configs der entfernten Container | gelöscht |
| `~/config.yaml` (Home-Root, mit Platzhaltertexten) | unbenutzte Vorlage, nicht die echte Dashboard-Config (die liegt unter `~/pidashboard/app/config.yaml`) | gelöscht |
| `docker`-Zeilen/Aliase in `~/.bashrc` (Login-Banner) | zeigten auf nicht mehr existierendes Programm | gezielt per `sed` entfernt, Rest des Banners bleibt |

Ergebnis: Pi ist jetzt sauber, nur noch Standard-Raspberry-Pi-OS + `pidash` (eigenes Projekt, bleibt bestehen).

## Phase 0 — Status: abgeschlossen ✅


**Nachtrag (17.08.2026):** `pidash` nachträglich ebenfalls entfernt (Dienst, Sudoers-Regel, App-Ordner) — Dominik wollte einen wirklich einheitlichen, sauberen Pi nur für dieses Projekt. Quellcode bleibt unangetastet auf dem Mac (`~/Desktop/Claude/PiDashboard`).
