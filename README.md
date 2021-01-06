# Raspberry Setup

## Sistema Operativo
Che si abbia il file con il SO o meno si può usare Raspberry Pi Imager scaricabile a [questo link](https://www.raspberrypi.org/software/). Il sistema operativo "standard" per Raspberry è `Raspberry Pi OS`.

## Setup iniziale senza monitor

Se si aggiunge al root un file vuoto chiamato `SSH` (senza estensione), verrà abilitato l'SSH e si potrà accedere con le credenziali di default: username `pi` e password `raspberry`. Quest'ultima si potrà cambiare con il comando `sudo raspi-config`.  

Inoltre si può configurare il wifi aggiungendo un file chiamato `wpa_supplicant.conf` con il seguente testo

```conf
country=IT
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    scan_ssid=1
    ssid="YOUR_WIFI_SSID"
    psk="YOUR_WIFI_PASSWORD"
}
```

Per la connessione tramite SSH: `ssh pi@raspberrypi.local`. 

Per sapere l'indirizzo IP si può lanciare il comando `hostname -I`

Per sapere il percorso delle chiavette USB, guardare mountpoint dal `lsblk`

Primo comando per eseguire gli aggiornamenti: `sudo apt-get update && sudo apt-get upgrade`.

## Connessione VNC
Per abilitare la connessione tramite VNC lanciare il comando `sudo raspi-config`. Selezionare ora `3. Interface Options` > `P3 VNC` > `<Yes>` > `<Ok>` > `<Finish>`.

Per collegarsi scaricare VNC viewer dal [seguente link](https://www.realvnc.com/en/connect/download/viewer/) e aggiungere una nuova connessione con la voce `VCN Server` riempita con `raspberrypi.local`. Facendo doppio click su questa connessione verranno chieste le credenziali.

## Node.js
Lanciare i comandi
```sh
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
sudo apt-get -y install nodejs
```

## OpenMediaVault
Per installare lanciare il seguente comando: `wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash` (ci impiegherà vari minuti).

Consigliato il reboot: `sudo reboot`. 

Il servizio è ora raggiungibile sulla porta `80`, con le credenziali di default: username `admin`, password: `openmediavault`. 

Setup iniziale:
- In `Impostazioni Generali` si può cambiare la porta (es. impostandola ad `8080`)
- In `Impostazioni Generali` si può cambiare il timeout per la disconnessione (default 5 minuti)
- In `Data e Ora` si può impostare il fusorario corretto, cioè `Europe/Rome`
- In `Utente` si può modificare la password 
- In `Rete` si può modificare il nome dell'host, del dominio e impostare l'indirizzo IP statico nella scheda `Interfacce`

Consigliato installare `openmediavault-fail2ban` nella sezione `Plugins`

Si possono installare docker, docker-compose e portainer nella sezione `OMV-Extras`. Aprire la scheda `Docker`, e cliccando sul menù a cascata di `Docker` e `Portainer` si potranno installare con la voce `Installa`. Con questa stessa modalità si possono poi disinstallare.

Se installato Portainer sarà raggiungibile alla porta `9000`. Le credenziali verranno impostate alla prima connessione. Scegliere Docker come container environment.

## Docker e DockerCompose
Se non si è installato tramite OpenMediaVault lanciare i seguenti comandi:
```sh
curl -sSL https://get.docker.com | bash
sudo usermod -aG docker pi
sudo apt-get install -y libffi-dev libssl-dev python3 python3-pip python3-setuptools
sudo apt install python3-dev
sudo pip3 install docker-compose
```

## PiHole
### Installazione normale
Lanciare il comando `curl -sSL https://install.pi-hole.net | bash`

### Installazione tramite docker-compose
Di seguito il `docker-compose.yml` per usare PiHole in un container Docker. 
```yml
version: "2"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:v5.1.2
    restart: unless-stopped
    environment:
      - TZ=Europe/Rome
      - WEBPASSWORD=WEB_ACCESS_PASSWORD
      - DNS1=8.8.8.8
      - DNS2=8.8.4.4
      - DNSSEC=true
    volumes:
      - /home/pi/pihole/data:/etc/pihole/
      - /home/pi/pihole/dnsmasq.d/:/etc/dnsmasq.d/
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 80:80 # Se la porta 80 è occupata da un altro servizio, cambiare in TUA_PORTA:80
    cap_add:
      - NET_ADMIN
```

Se c'è un conflitto sulla porta `53` si possono lanciare i seguenti comandi
```sh
sudo systemctl disable systemd-resolved.service
sudo systemctl stop systemd-resolved
```

## OpenVPN (o WireGuard)
Per installare OpenVPN (o Wireguard) lanciare il seguente comando: `curl -L https://install.pivpn.io | bash`

TODO: passaggi da fare

## Home Assistant
Di seguito il file `docker-compose.yml` per usare PiHole in un container Docker.
```yml
version: "2"

services:
  homeassistant:
    container_name: homeassistant
    image: homeassistant/raspberrypi4-homeassistant:stable # Per Pi3: homeassistant/raspberrypi3-homeassistant:stable
    restart: unless-stopped
    environment:
      - TZ=Europe/Rome
    volumes:
      - /home/pi/homeassistant/config:/config
      - /home/pi/homeassistant/media:/media
    ports:
      - 8123:8123
```

Come si può vedere, il servizio sarà raggiungibile alla porta `8123`. Alla prima connessione verrà chiesto di impostare le credenziali.

TODO: la versione 2.0 è la stessa per portainer?

## Plex
Di seguito il file `docker-compose.yml` per usare Plex in un container Docker. Si deve sostituire il valore `YOUR_PLEX_CLAIM_CODE` con il valore ottenuto a [questo link](https://www.plex.tv/claim) dopo aver eseguito l'accesso.
```yml
version: "2"

services:
  plex:
    container_name: plex
    image: ghcr.io/linuxserver/plex:latest
    restart: unless-stopped
    environment:
      - PUID=998
      - PGID=100
      - VERSION=docker
      - UMASK_SET=022
      - PLEX_CLAIM=YOUR_PLEX_CLAIM_CODE
    network_mode: host
    volumes:
      - /home/pi/plex/config:/config
      - /home/pi/plex/tv:/tv
      - /home/pi/plex/movies:/movies
    ports:
      - 32400:32400
```
Come si può vedere, il servizio sarà raggiungibile alla porta `32400`.

## Transmission
Di seguito il file `docker-compose.yml` per usare Transmission in un container Docker.
```yml
version: "2"

services:
  transmission:
    container_name: transmission
    image: ghcr.io/linuxserver/transmission
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Rome
      - TRANSMISSION_WEB_HOME=/combustion-release/ #optional
      - USER=username #optional
      - PASS=password #optional
    volumes:
      - /home/pi/transmission/config:/config
      - /home/pi/transmission/downloads:/downloads
      - /home/pi/transmission/watch:/watch
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
```