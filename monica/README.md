# Monica CRM
[Monica](https://www.monicahq.com/) is a personal CRM tool.

This installation is based on the FPM example [provided in the MonicaHQ README](https://github.com/monicahq/docker/tree/main#fpm-version).

Specifically, we have a [Docker Compose](./docker-compose.yml) file that creates the following containers

1. monica-app, which is the FPM instance of monica.
2. monica-db, which is a MySQL instance to power the backend.
3. monica-web, which is the nginx instance that serves the monica FPM content via HTTP (not https).

We rely on the DSM software for the following:

1. Dynamic DNS functionality
2. Managing the docker containers
3. Creating a reverse proxy
4. HTTPS (including certificate management)
5. Backing up storage

You are going to need to figure out your own solution for SMTP if you want Monica to be able to send emails.  You might consider using [Amazon SES](https://aws.amazon.com/ses/) which seems to cost 10 cents a month for up to 1000 emails.

## Installing

### Setting up your DSM
1. Install the Docker package, which can be found and installed via the DSM Package Manager application.

2. Create a folder at `/volume1/docker/monica/storage` (see [Folder locations](####folder-locations)). This stores static assets associated with your Monica instance -- things like photos uploaded for your contacts.

3. Create a folder at `/volume1/docker/monica/mysql` (see [Folder locations](####folder-locations)).  This is where the database files associated with this Monica installation as well -- this will prevent you from losing all of your data when you update your Docker images over time.

#### Folder locations

In the DSM File Manager the `volume1` seems to be omitted from the path.  I believe it just means these directory structures need to be on the first configured storage pool.  If you want a different location for either of these folders just update the respective `volumes:` sections in the `docker-compose.yml`.

### Setting up your `.env` file

1. Generate a random key for use later
```bash
echo -n 'base64:'; openssl rand -base64 32
```

2. Populate the `.env` file:

```bash
cp .env.template .env
vi .env
```

3. Upload this folder to Synology (e.g. to `/home/dsm/monica`)

4. SSH into Synology

5. cd to the location of the folder uploaded in step 3. (e.g. `cd /home/dsm/monica`)

6. Invoke docker compose

```bash
docker-compose up -d
```

This will take a while to spin up everything and run migrations; you can use the DSM to view the logs (or I suppose the docker CLI if you're fancy).

### (Optional) Setting up Dynamic DNS / a host name
The instructions above will allow you to access an `http` version of monica by navigating directly to `http://your.dsm.ip.address:4000`.  However, if you want to be able to access Monica outside of your LAN (and using https) you'll need to use DSM to set up dynamic DNS (DDNS) and a reverse proxy.

1. Assuming you want to be able to access Monica outside of your LAN and using HTTPS, you will want to set up a dynamic DNS service.  Synology provides one for free, but even if you use a third party service the [instructions are generally the same](https://kb.synology.com/en-us/DSM/help/DSM/AdminCenter/connection_ddns).  This would give you something like `http://username.synology.me`

3. If you plan to run multiple services on your DSM, you should create a CNAME DNS record on a separate domain name.  For instance `monica.yourdomain.com`.  The value for this record should be your DDNS domain (e.g. `username.synology.me`)

4. Use DSM to [set up an HTTPS certificate](https://kb.synology.com/en-nz/DSM/help/DSM/AdminCenter/connection_certificate) for the specified subdomain.

5. Use the DSM's Synology Login Portal to create a [reverse proxy](https://kb.synology.com/en-ca/DSM/help/DSM/AdminCenter/system_login_portal_advanced) that routes your custom domain name to port 4000.

Values for proxy rules:

```
Reverse Proxy Name: Monica (HTTPS) <-- this can be whatever you want

SOURCE
Protocol: HTTPS
Hostname: monica.yourdomain.com
Port: 443
Enable HSTS: no
Access control profile: Not configured <-- you can limit access to specific IPs if you want to be extra safe

DESTINATION
Protocol: HTTP
Hostname: localhost
Port: 4000
```

