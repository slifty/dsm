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

You are going to need to figure out your own soluton for SMTP if you want Monica to be able to send emails.  You might consider using [Amazon SES](https://aws.amazon.com/ses/) which seems to cost 10 cents a month for up to 1000 emails.

## Installing

### Setting up your DSM
1. You will need the Docker package, which can be installed via the DSM Package Manager


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
