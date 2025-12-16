# Hetzner DynDNS (Cloud API)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker Pulls](https://img.shields.io/docker/pulls/mbaiti/hetzner-dyndns.svg)](https://hub.docker.com/r/mbaiti/hetzner-dyndns)

Ein schlanker Dynamic DNS Updater für Hetzner DNS-Einträge, der die neue Hetzner Cloud API verwendet. Dieses Tool überwacht deine öffentliche IP-Adresse und aktualisiert automatisch einen spezifischen A-Record in deiner Hetzner DNS-Zone, falls sich die IP-Adresse ändert. Ideal für Heimserver oder andere dynamische IP-Umgebungen.

Dieses Projekt ist eine angepasste Version des ursprünglichen `filiparag/hetzner_ddns`, wurde aber komplett auf den Betrieb in einem Docker-Container mit Umgebungsvariablen optimiert.

## Features

*   **Verwendet die neue Hetzner Cloud API:** Kompatibel mit der aktuellen Hetzner DNS-Verwaltung.
*   **Containerisiert:** Läuft zuverlässig und isoliert in einem Docker-Container.
*   **Konfiguration über Umgebungsvariablen:** Einfache und sichere Konfiguration mit `docker-compose`.
*   **Automatische IP-Erkennung:** Prüft regelmäßig die öffentliche IPv4-Adresse.
*   **Minimalistisch:** Schlankes Alpine Linux-Basis-Image und reines Shell-Skript für geringen Ressourcenverbrauch.
*   **Zuverlässig:** Aktualisiert den DNS-Eintrag nur bei IP-Änderung.

## Hintergrund zur API-Umstellung

Hetzner hat die Verwaltung seiner DNS-Zonen in die Hetzner Cloud API integriert. Die alte dedizierte Hetzner DNS Console API ist nicht mehr die empfohlene Methode. Dieses Skript wurde entwickelt, um diese neue API-Struktur zu unterstützen. Dies erfordert die Verwendung eines API-Tokens aus der Hetzner Cloud Konsole und angepasste API-Endpunkte für die DNS-Verwaltung.

## Voraussetzungen

*   Docker und Docker Compose installiert
*   Ein Hetzner Cloud Konto
*   Eine DNS-Zone, die in die Hetzner Cloud Konsole migriert wurde (falls sie ursprünglich in der DNS Console erstellt wurde).

## Einrichtung

### API Token generieren

1.  Melde dich in deiner [Hetzner Cloud Konsole](https://console.hetzner.cloud/) an.
2.  Navigiere zu "Access" -> "API Tokens".
3.  Erstelle einen neuen API Token. Gib ihm einen sprechenden Namen (z.B. "hetzner-dyndns").
4.  Stelle sicher, dass der Token mindestens die Berechtigung zum **Lesen und Schreiben von DNS-Records** hat.
5.  Kopiere den generierten Token. Er wird für die Umgebungsvariable `HETZNER_CLOUD_API_TOKEN` benötigt.

### DNS Zone und Record vorbereiten

1.  Stelle sicher, dass die DNS-Zone, die du aktualisieren möchtest, in der Hetzner Cloud Konsole sichtbar und verwaltbar ist. Falls nicht, musst du sie eventuell manuell migrieren.
2.  Erstelle einen A-Record für den Hostnamen, den du aktualisieren möchtest (z.B. `myhome.example.com` oder `@` für die Domain selbst), und gib ihm einen Platzhalter-Wert (z.B. `127.0.0.1`). Das Skript wird diesen Wert später überschreiben.

### Docker-Compose Beispiel 

```yaml
services:
  hetzner-dyndns:
    image: mbaiti/hetzner-dyndns:latest
    container_name: hetzner-dyndns
    restart: unless-stopped
    environment:
      - HETZNER_CLOUD_API_TOKEN=dein_hetzner_cloud_api_token
      - HETZNER_DNS_ZONE_NAME=deine_domain.com #Der Name deiner DNS-Zone (z.B. "example.com")
      - HETZNER_DNS_RECORD_NAME=subdomain_oder_@ #z.B. "myhost" für myhost.deine_domain.com oder "@" für deine_domain.com
      - CHECK_INTERVAL_SECONDS=300
```

### Logging
Das Skript gibt Logs mit Zeitstempeln und dem Status der Operationen aus.

* **INFO:** Für normale Operationen und erfolgreiche Updates.
* **DEBUG:** Detailliertere Informationen (z.B. wenn keine IP-Änderung erkannt wird). Wird in der aktuellen Version nicht verwendet, könnte aber bei Bedarf erweitert werden.
* **WARNING:** Für nicht-kritische Probleme (z.B. IP-Ermittlung fehlgeschlagen).
* **ERROR:** Für kritische Fehler (z.B. API-Fehler, fehlende Konfiguration).
