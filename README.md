# docker-caddy-mediawiki

Mediawiki running in Docker, set up for use with a Caddy reverse proxy.

This project uses Docker Compose to run the components needed by Mediawiki:

* Caddy as the webserver
* PHPFPM as the PHP server
* Mariadb as the database

I am using PHP 8.1 and it's not working 100% with the released version of mediawiki,
I am going to live with it until mediawiki catches up. It is easier than backpedaling to PHP 7.

I use Caddy as the webserver because I am using a Caddy reverse proxy.
I am trying to keep things Caddy based. No real reason for this. I've also used
the mediawiki docker which is based on Apache and I've run one on nginx.
They all work fine.

## Deployment

```bash
cp sample.env .env
vi .env
docker-compose build
docker-compose up -d
```

## Moving to a new server

I am changing service providers. The old service was hosted and the new one is VPS,
so I have to move all settings, files, and the database.

There are upgrade notes here.
https://www.mediawiki.org/wiki/Manual:Upgrading

### Settings

Create a new LocalSettings.php file by running the Wiki installation process.
Then you can copy the old one to the new machine and compare them and make appropriate edits.

Install the edited LocalSettings.php into the volume with

```bash
docker cp LocalSettings.php phpfpm:/srv
```

### The images/ folder (file storage)

On my new set up (the one described here) I keep the images
in a separate Docker volume. No particular reason, it's just how I organized things.

```bash
docker cp images phpfpm:/srv
```

### Dump the database

You have to know your username and password and the name of the database. They are all exposed in the LocalSettings.php.

Dump the database to a SQL file and then copy it to the new machine.

```bash
mysqldump -u wildsong_wiki -p=IjihBq5FRYL3zX8j wildsong_wiki > old_wiki.sql
scp old_wiki.sql NEWMACHINE:
```

### Load the database

I did this by creating a docker that had access to the SQL dump file, and could
connect to the running database over the 'database' virtual network.

```bash
docker run --rm -it -v `pwd`/old_wiki.sql:/tmp/old_wiki.sql --network=database mariadb:latest bash
mysql -u wikiuser -p -h database wiki < /tmp/old_wiki.sql 
```

This works but the old wiki ran on Mediawiki 1.30 and the latest is 1.38.4
which means they are not compatible. 

### Run update

```bash
docker exec -it phpfpm sh
cd /srv/maintenance
php update.php
```

I went from 1.30 to the latest (1.38.4) and it worked. Took 15s on Bellman, on 24s Tektonic.

## Resources

See the reverse proxy that I use with this,
https://github.com/Wildsong/docker-caddy-proxy

