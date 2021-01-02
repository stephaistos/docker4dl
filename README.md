# docker4dl
docker-compose file for setting up common torrent download tools behind VPN (radarr/sonarr/jackett/deluge/gluetun/flaresolverr)

General idea is to have access to all the tools through web interface while they all run on separate docker containers using a VPN.
Created for my Synology NAS, extra details will be given for that specific case.

# In details:
Separate docker containers for each service, all using gluetun container (VPN) for network access.

- [radarr](https://github.com/Radarr/Radarr) : movie collection manager
- [sonarr](https://github.com/Sonarr/Sonarr) : tvshow collection manager
- [jackett](https://github.com/Jackett/Jackett) : bitTorrent indexer
- [flaresolverr](https://github.com/FlareSolverr/FlareSolverr) : required for some jackett indexer login
- [deluge](https://github.com/deluge-torrent/deluge) : bitTorrent client to download content
- [gluetun](https://github.com/qdm12/gluetun) : lightweight vpn client

Based on [wbollock work](https://github.com/wbollock/Radarr-and-Sonarr-Setup/blob/master/Docker-Compose_Setup.md)


# Prerequisite
- Own VPN provider account supported by gluetun (currently supported: Private Internet Access, Mullvad, Windscribe, Surfshark Cyberghost, VyprVPN, NordVPN, PureVPN and Privado VPN servers)

If you don't have a VPN yet, an exhaustive comparison chart can be found [here](https://www.safetydetectives.com/best-vpns/)

[Mullvad](https://mullvad.net) will be used as an example.

- ssh terminal access to your server/NAS and root privileges
For Synology NAS, install Docker via Package Center.
Enable SSH service via Control Panel/Terminal & SNMP

- docker and docker-compose installed

To check:

`docker -v`
and
`docker-compose --version`


# Variables needed and how to get them

Open ssh connection to server/NAS.
```
PUID : use `id -u` command in terminal

PGID : use `id -g` command in terminal

USER : 'whoami'

USERGROUP : 'id -gn'

TZ : your time zone, find it [here](https://docs.diladele.com/docker/timezones.html)

SERVER_IP : use ifconfig to find interface and ip
```

Example: 
```
`id` -> uid=1026(bob) gid=100(users) groups=100(users),101(administrators)

PUID=1026

PGID=100

USER=bob

USERGROUP=users

TZ=Hongkong

SERVER_IP=192.168.1.210
```

# Folders need to be created for the different services:

Config folders examples:
```
RADARR_CONFIG_PATH=/etc/radarr/

SONARR_CONFIG_PATH=/etc/sonarr/

DELUGE_CONFIG_PATH=/etc/deluge/

JACKETT_CONFIG_PATH=/etc/jackett/

OPENVPN_CONFIG=/etc/openvpnconfig/
```

Downloads folders examples:
```
DOWNLOAD_FOLDER=/volume1/Torrents/Downloads_Deluge   <- This is where deluge will download the content, it is temporary and once the download is finished it will be moved to one of the next 2 folders.

FILMS=/volume1/Torrents/Films

TVSHOWS=/volume1/Torrents/Tvshows
```

Once these folders are created, make sure they have the right ownership.

`sudo chown -R USER:USERGROUP RADARR_CONFIG_PATH SONARR_CONFIG_PATH DELUGE_CONFIG_PATH JACKETT_CONFIG_PATH OPENVPN_CONFIG`

Example:
`sudo chown -R bob:users /etc/radarr/ /etc/deluge/ /etc/sonarr/ /etc/jackett/ /etc/openvpnconfig/`


# OpenVPN configuration for gluetun container

The configuration will depend on the VPN provider, please refer to the beautiful [gluetun wiki for environement variables](https://github.com/qdm12/gluetun/wiki/Environment-variables)

Example Mullvad VPN:
- OPENVPN_USER=100011000100 # this is your Mullvad account number
- OPENVPN_PASSWORD=m # for Mullvad this is always just 'm'
- VPNSP=mullvad
- COUNTRY=Singapore # change the country based on your location/needs
- CITY=Singapore
- ISP=M247


# Creating docker-compose file

Use the template and replace with your variables where needed.

Rename the file docker-compose.yml

Run `sudo docker-compose config` in the folder where the file is.

If there's no mistake (identation errors/use spaces not tabs), it will print the file.

Then run 'sudo docker-compose up -d'

`sudo docker ps -a` will show the status of the containers.

In case there's a problem, try `sudo docker logs CONTAINER_ID` to see what's going on.


# Is it working?

## Checking gluetun vpn container status:

`docker run --rm --network=container:gluetun alpine:3.12 wget -qO- https://ipinfo.io` [source](https://github.com/qdm12/gluetun/wiki/Test-your-setup)

It's possible to open web browser and go to page:

`http://SERVER_IP:8000/v1/openvpn/status`

Example: 

`http://192.168.1.210:8000/v1/openvpn/status`

Current settings are acessible using `http://SERVER_IP:8000/v1/openvpn/settings`

Other options available, see : [gluetun http control server](https://github.com/qdm12/gluetun/wiki/HTTP-Control-server)

If the gluetun container does not seem to work properly, I suggest trying to debug it alone first using information provided on the github project page.

## Checking and configuring services:

### Deluge:

Visit `http://SERVER_IP:8112/`

Change password from default `deluge`

Under Preferences/Downloads

Set Downloads to : `/downloads`

### Jackett:
Visit `http://SERVER_IP:9117/`

Set FlareSolverr API URL: `http://localhost:8191`

### Sonarr:
Visit `http://SERVER_IP:8989/`

Go to Settings/Download client

Add Download client (Torrent) - Deluge

Host: localhost

Port: 8112

Don't forget to add Indexers from Jackett or others.

### Radarr:

Visit `http://SERVER_IP:7878/`

Go to Settings/Download client

Add Deluge (similar to what was done for Sonarr) 

Add indexers.



