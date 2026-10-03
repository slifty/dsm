# Navidrome
[Navidrome](https://www.navidrome.org/) is an open source web-based music collection server and streamer, compatible with Subsonic/Airsonic clients.

This installation is based on their [Docker installation documentation](https://www.navidrome.org/docs/installation/docker/).

Specifically, we have a [Docker Compose](./docker-compose.yml) file that creates the following container:

1. navidrome-app, which is the navidrome application (includes an embedded database).

We rely on the DSM software for the following:

1. Dynamic DNS functionality
2. Managing the docker containers
3. Creating a reverse proxy
4. HTTPS (including certificate management)
5. Backing up storage

## Installing

### Setting up your DSM
1. Install the Docker package, which can be found and installed via the DSM Package Manager application.

2. Create a folder at `/volume1/docker/navidrome/data` (see [Folder locations](####folder-locations)).

#### Folder locations

In the DSM File Manager the `volume1` seems to be omitted from the path.  I believe it just means these directory structures need to be on the first configured storage pool.  If you want a different location for either of these folders just update the respective `volumes:` sections in the `docker-compose.yml`.

### Setting up your `.env` file

1. Populate the `.env` file:

```bash
cp .env.template .env
vi .env
```

2. Set `MUSIC_PATH` to the path of your existing music library on the Synology (e.g. `/volume1/music`).  This directory will be mounted read-only into the container.

3. Upload this folder to Synology (e.g. to `/home/dsm/navidrome`)

4. SSH into Synology

5. cd to the location of the folder uploaded in step 3. (e.g. `cd /home/dsm/navidrome`)

6. Invoke docker compose

```bash
docker-compose up -d
```

### (Optional) Setting up Dynamic DNS / a host name
The instructions above will allow you to access an `http` version of navidrome by navigating directly to `http://your.dsm.ip.address:4005`.  However, if you want to be able to access Navidrome outside of your LAN (and using https) you'll need to use DSM to set up dynamic DNS (DDNS) and a reverse proxy.

1. Assuming you want to be able to access Navidrome outside of your LAN and using HTTPS, you will want to set up a dynamic DNS service.  Synology provides one for free, but even if you use a third party service the [instructions are generally the same](https://kb.synology.com/en-us/DSM/help/DSM/AdminCenter/connection_ddns).  This would give you something like `http://username.synology.me`

3. If you plan to run multiple services on your DSM, you should create a CNAME DNS record on a separate domain name.  For instance `navidrome.yourdomain.com`.  The value for this record should be your DDNS domain (e.g. `username.synology.me`)

4. Use DSM to [set up an HTTPS certificate](https://kb.synology.com/en-nz/DSM/help/DSM/AdminCenter/connection_certificate) for the specified subdomain.

5. Use the DSM's Synology Login Portal to create a [reverse proxy](https://kb.synology.com/en-ca/DSM/help/DSM/AdminCenter/system_login_portal_advanced) that routes your custom domain name to port 4005.

Values for proxy rules:

```
Reverse Proxy Name: Navidrome (HTTPS) <-- this can be whatever you want

SOURCE
Protocol: HTTPS
Hostname: navidrome.yourdomain.com
Port: 443
Enable HSTS: no
Access control profile: Not configured <-- you can limit access to specific IPs if you want to be extra safe

DESTINATION
Protocol: HTTP
Hostname: localhost
Port: 4005
```
