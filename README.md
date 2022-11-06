# docker-caddy-mediawiki

Mediawiki running in Docker, set up for use with a Caddy reverse proxy.

This project uses Docker Compose to run the components needed by Mediawiki:

* Caddy as the webserver
* PHPFPM as the PHP server
* Mariadb as the database

I use Caddy as the webserver because I am using a Caddy reverse proxy.
I am trying to keep things Caddy based. No real reason for this. I've also used
the mediawiki docker which is based on Apache and I've run one on nginx.
They all work fine.

## Moving to a new server

Create a new LocalSettings.php file by running the Wiki installation process.
Then you can copy the old one to the new machine and compare them and make appropriate edits.

### Copy the images/ folder. 

On my new set up (the one described here) I keep the images
in a separate Docker volume. No particular reason, it's just how I organized things.

### Dump the database

You have to know your username and password.

```bash
mysqldump -u wildsong_wiki -p=IjihBq5FRYL3zX8j wildsong_wiki > old_wiki.sql
scp old_wiki.sql NEWMACHINE:

### Copy the database to the new machine and load it.

## Resources

See the reverse proxy that I use with this,
https://github.com/Wildsong/docker-caddy-proxy

