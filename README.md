# portainer-cheat-sheet

## Install

### Docker

#### Linux
- https://docs.portainer.io/start/install-ce/server/docker/linux


# Portainer CE mit Docker auf Linux installieren

## Zweck

Diese Anleitung installiert **Portainer Community Edition (CE)** als Docker-Container auf einem Linux-System.

Portainer besteht aus:

- **Portainer Server**
- optional: **Portainer Agent**

Für eine lokale Docker-Standalone-Installation brauchst du nur den **Portainer Server**.

---

## Voraussetzungen

Du brauchst:

- Docker installiert und funktionierend
- `sudo`-Zugriff auf dem Host
- Zugriff auf Docker über den Unix-Socket `/var/run/docker.sock`

Portainer nutzt standardmäßig:

- `9443` für die Weboberfläche über HTTPS
- `8000` für den TCP-Tunnel-Server, nur nötig für Edge-Agent-Features
- optional `9000` für altes HTTP / Legacy-Zugriff

Hinweis:

Auf Ubuntu sollte Docker möglichst **nicht über Snap** installiert sein, weil es dabei Kompatibilitätsprobleme geben kann.

---

# Variante 1: Installation mit `docker run`

## 1. Docker-Volume erstellen

```bash
docker volume create portainer_data
````

## 2. Portainer Server starten

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:sts
```

## Optional: Port 9000 zusätzlich öffnen

Nur falls du den alten HTTP-Port brauchst:

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  -p 9000:9000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:sts
```

---

# Variante 2: Installation mit Docker Compose

## 1. Compose-Datei herunterladen

```bash
curl -L https://downloads.portainer.io/ce-sts/portainer-compose.yaml -o portainer-compose.yaml
```

## Alternative: Compose-Datei manuell erstellen

Datei erstellen:

```bash
nano portainer-compose.yaml
```

Inhalt:

```yaml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:sts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - 9443:9443
      - 8000:8000 # Remove if you do not intend to use Edge Agents

volumes:
  portainer_data:
    name: portainer_data

networks:
  default:
    name: portainer_network
```

## 2. Portainer mit Docker Compose starten

```bash
docker compose -f portainer-compose.yaml up -d
```

---

# Prüfen, ob Portainer läuft

```bash
docker ps
```

Du solltest ungefähr so etwas sehen:

```text
CONTAINER ID   IMAGE                        COMMAND        STATUS         PORTS                     NAMES
xxxxxxxxxxxx   portainer/portainer-ce:sts   "/portainer"   Up             0.0.0.0:9443->9443/tcp    portainer
```

---

# Portainer öffnen

Im Browser öffnen:

```text
https://localhost:9443
```

Wenn Portainer auf einem anderen Rechner läuft:

```text
https://SERVER-IP:9443
```

Beispiel:

```text
https://192.168.178.50:9443
```

---

# Wichtige Hinweise

## Self-signed SSL-Zertifikat

Portainer erzeugt standardmäßig ein selbstsigniertes SSL-Zertifikat für Port `9443`.

Deshalb kann der Browser eine Warnung anzeigen.

Das ist bei lokaler Nutzung normal.

---

## Port 8000

Port `8000` brauchst du nur, wenn du Portainer Edge Agents / Edge Compute verwenden willst.

Wenn du nur lokale Docker-Container überwachen willst, kannst du `8000` theoretisch weglassen.

Dann wäre der `docker run`-Befehl:

```bash
docker run -d \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:sts
```

Oder in Docker Compose:

```yaml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:sts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - 9443:9443

volumes:
  portainer_data:
    name: portainer_data

networks:
  default:
    name: portainer_network
```

---

# Nützliche Befehle

## Logs anzeigen

```bash
docker logs portainer
```

Live-Logs:

```bash
docker logs -f portainer
```

## Container stoppen

```bash
docker stop portainer
```

## Container starten

```bash
docker start portainer
```

## Container entfernen

```bash
docker rm -f portainer
```

## Volume anzeigen

```bash
docker volume ls
```

## Portainer-Daten-Volume anzeigen

```bash
docker volume inspect portainer_data
```

---

# Empfehlung für lokale Überwachung

Wenn du Portainer nur als grafische Oberfläche für deine lokalen Docker-Container willst, reicht normalerweise:

```bash
docker volume create portainer_data
```

```bash
docker run -d \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:sts
```

Dann öffnen:

```text
https://localhost:9443
```

```
::contentReference[oaicite:1]{index=1}
```

[1]: https://docs.portainer.io/start/install-ce/server/docker/linux "Install Portainer CE with Docker on Linux | Portainer Documentation"
