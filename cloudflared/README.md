# Cloudflared
[Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) lets you serve services on the DSM to the internet without opening any ports on your router.  `cloudflared` runs on the DSM and makes an outbound connection to Cloudflare, and Cloudflare sends requests for your hostnames down that connection.

This is safer than forwarding ports 80 and 443 to the DSM, which exposes the DSM's entire web server (not just the services you meant to share) to anyone who scans your IP address.

We have a [Docker Compose](./docker-compose.yml) file that creates a single `cloudflared` container.  It uses host networking so that it can reach the other services on the DSM at `localhost` (see the [port mapping](../README.md#internal-service--port-mapping)).

This requires that your domain's DNS is managed by Cloudflare.  Tunnels are included in Cloudflare's free plan.

## Installing

### Creating the tunnel
1. In the Cloudflare dashboard go to Zero Trust -> Networks -> Tunnels and create a tunnel of type "Cloudflared".

2. Copy the tunnel token from the install instructions it shows you (the long string after `--token`).

### Setting up your `.env` file

1. Populate the `.env` file with the tunnel token:

```bash
cp .env.template .env
vi .env
```

2. Upload this folder to Synology (e.g. to `/home/docker/cloudflared`)

3. SSH into Synology

4. cd to the location of the folder uploaded in step 2. (e.g. `cd /home/docker/cloudflared`)

5. Invoke docker compose

```bash
docker-compose up -d
```

The tunnel should show as "Healthy" in the Cloudflare dashboard within a minute or so.

### Routing a hostname
For each service you want to expose, add a public hostname to the tunnel in the Cloudflare dashboard.  Point it directly at the service's port rather than at the DSM's reverse proxy, so that nothing else on the DSM is reachable through the tunnel.

For example, for Vaultwarden:

```
Subdomain: vaultwarden
Domain: yourdomain.com

SERVICE
Type: HTTP
URL: localhost:4001
```

Cloudflare will create the DNS record for you.  If a record for that hostname already exists (e.g. a CNAME to your DDNS domain) you will need to delete it first.

Once the service works through the tunnel, remove any port forwards for it on your router and the matching reverse proxy rule in DSM.
