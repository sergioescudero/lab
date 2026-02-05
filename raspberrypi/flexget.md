tasks:
  # downloading task and remove finished torrents
  # called via cron every 30 minutes.
  download-showrss:
    priority: 1
    rss: http://showrss.info/user/38243.rss
    all_series: yes
    # use transmission to download the torrents
    transmission:
      host: localhost
      port: 9091
      username: 'sarkis'
      password: 'uleila.'

/home/dietpi/.flexget/config.yml



2
3
4
5
sudo apt-get install transmission-daemon python-pip
sudo pip install flexget
sudo pip install transmissionrpc
sudo pip install periscope
mkdir /home/pi/.config



sudo cat /etc/transmission-daemon/settings.json
{
    "alt-speed-down": 50,
    "alt-speed-enabled": false,
    "alt-speed-time-begin": 540,
    "alt-speed-time-day": 127,
    "alt-speed-time-enabled": false,
    "alt-speed-time-end": 1020,
    "alt-speed-up": 50,
    "bind-address-ipv4": "0.0.0.0",
    "bind-address-ipv6": "::",
    "blocklist-enabled": false,
    "blocklist-url": "http://www.example.com/blocklist",
    "cache-size-mb": 4,
    "dht-enabled": true,
    "download-dir": "/home/shares/public/media/downloads/completed",
    "download-limit": 1000000000,
    "download-limit-enabled": false,
    "download-queue-enabled": false,
    "download-queue-size": 55,
    "encryption": 1,
    "idle-seeding-limit": 30,
    "idle-seeding-limit-enabled": false,
    "incomplete-dir": "/home/shares/public/media/downloads/incompleted",
    "incomplete-dir-enabled": true,
    "lpd-enabled": false,
    "max-peers-global": 200,
    "message-level": 1,
    "peer-congestion-algorithm": "",
    "peer-id-ttl-hours": 6,
    "peer-limit-global": 200,
    "peer-limit-per-torrent": 50,
    "peer-port": 51413,
    "peer-port-random-high": 65535,
    "peer-port-random-low": 49152,
    "peer-port-random-on-start": false,
    "peer-socket-tos": "default",
    "pex-enabled": true,
    "port-forwarding-enabled": false,
    "preallocation": 1,
    "prefetch-enabled": true,
    "queue-stalled-enabled": true,
    "queue-stalled-minutes": 30,
    "ratio-limit": 2,
    "ratio-limit-enabled": false,
    "rename-partial-files": true,
    "rpc-authentication-required": true,
    "rpc-bind-address": "0.0.0.0",
    "rpc-enabled": true,
    "rpc-host-whitelist": "",
    "rpc-host-whitelist-enabled": false,
    "rpc-password": "{c7c2a89bdc4883f3891b39716aa56930764f0b5eQXHKBHmb",
    "rpc-port": 9091,
    "rpc-url": "/transmission/",
    "rpc-username": "sarkis",
    "rpc-whitelist": "127.0.0.1",
    "rpc-whitelist-enabled": false,
    "scrape-paused-torrents-enabled": true,
    "script-torrent-done-enabled": true,
    "script-torrent-done-filename": "/home/dietpi/torrent-done.sh",
    "seed-queue-enabled": false,
    "seed-queue-size": 10,
    "speed-limit-down": 1410065,
    "speed-limit-down-enabled": false,
    "speed-limit-up": 100,
    "speed-limit-up-enabled": true,
    "start-added-torrents": true,
    "trash-original-torrent-files": false,
    "umask": 0,
    "upload-limit": 10,
    "upload-limit-enabled": true,
    "upload-slots-per-torrent": 14,
    "utp-enabled": true
}



dietpi@DietPi:~/.flexget$ cat /home/dietpi/torrent-done.sh
/usr/local/bin/flexget -l /var/log/flexget.log execute --tasks sort-shows


dietpi@DietPi:~/.flexget$ cat /home/dietpi/torrent-done.sh
/usr/local/bin/flexget -l /var/log/flexget.log execute --tasks sort-shows



#!/bin/bash
# ===============================
# Script automático de descarga de subtítulos en español
# Compatible con DietPi + FlexGet
# ===============================

# Carpeta donde se descargan las series y películas
SERIES_DIR="/home/shares/public/media/downloads/completed"

# Idioma de los subtítulos
LANG="es"

# Reemplazar subtítulos existentes? 1 = sí, 0 = solo si no existen
REPLACE_EXISTING=0

# Activar entorno virtual
source /home/dietpi/flexget-env/bin/activate

# Función para procesar una carpeta
process_folder() {
    local DIR="$1"
    echo "Procesando carpeta: $DIR"

    # Buscar archivos de vídeo
    find "$DIR" -type f \( -iname "*.mkv" -o -iname "*.mp4" -o -iname "*.avi" \) | while read -r FILE; do
        # Revisar si ya existe subtítulo
        if [ "$REPLACE_EXISTING" -eq 0 ] && ls "${FILE%.*}".*.srt 1> /dev/null 2>&1; then
            echo "Subtítulo ya existe para $FILE. Saltando."
            continue
        fi

        # Descargar subtítulos con subliminal
        echo "Descargando subtítulo para $FILE..."
        subliminal download -l "$LANG" "$FILE"
    done
}

# Procesar series y películas
process_folder "$SERIES_DIR"

echo "Subtítulos procesados correctamente."



