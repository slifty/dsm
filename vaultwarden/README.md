# Vaultwarden
[Vaultwarden](https://github.com/dani-garcia/vaultwarden) is a fully featured open source implementation of the BitWarden API. [BitWarden](https://bitwarden.com) is a password management tool.

This installation is based on their readme combined with their [configuration overview](https://github.com/dani-garcia/vaultwarden/wiki/Configuration-overview).

We have a [Docker Compose](./docker-compose.yml) file that creates the following containers:

1. `vaultwarden-app`, which is a single container that runs and serves vaultwarden.
2. `vaultwarden-db`, which is a PostgreSQL instance to power the backend.

We rely on the DSM software for the following:

1. Managing the docker containers
2. Back ups and storage

We rely on a [Cloudflare Tunnel](../cloudflared/README.md) for access from outside the LAN, including HTTPS.

You are going to need to figure out your own solution for SMTP for Vaultwarden to be able to send emails.
You might consider using [Amazon SES](https://aws.amazon.com/ses/) which costs 10 cents a month for up to 1000 emails.

## Installing

### Setting up your DSM
1. Install the Docker package, which can be found and installed via the DSM Package Manager application.

2. Create a folder at `/volume1/docker/vaultwarden/data` (see [Folder locations](####folder-locations)). This stores data associated with your Vaultwarden instance.

3. Create a folder at `/volume1/docker/vaultwarden/postgres/data` (see [Folder locations](####folder-locations)).  This is where the database files associated with this Vaultwarden installation as well -- this will prevent you from losing all of your data when you update your Docker images over time.

#### Folder locations

In the DSM File Manager the `volume1` seems to be omitted from the path.  I believe it just means these directory structures need to be on the first configured storage pool.  If you want a different location for either of these folders just update the respective `volumes:` sections in the `docker-compose.yml`.

### Setting up your `.env` file

1. Populate the `.env` file.  Set `DOMAIN` to the HTTPS host name you will route through the tunnel (e.g. `DOMAIN=https://vaultwarden.yourdomain.com`):

```bash
cp .env.template .env
vi .env
```

2. Upload this folder to Synology (e.g. to `/home/docker/vaultwarden`)

3. SSH into Synology

4. cd to the location of the folder uploaded in step 3. (e.g. `cd /home/docker/vaultwarden`)

5. Invoke docker compose

```bash
docker-compose up -d
```

This will take a while to spin up everything and run migrations; you can use the DSM to view the logs (or I suppose the docker CLI if you're fancy).

### Setting up a host name
Vaultwarden is only reachable through its host name.  Set up a [Cloudflare Tunnel](../cloudflared/README.md) and route a hostname (e.g. `vaultwarden.yourdomain.com`) to `localhost:4001`.

Don't forward ports on your router to the DSM for this; the tunnel doesn't need them, and forwarding them exposes the rest of the DSM to the internet.
