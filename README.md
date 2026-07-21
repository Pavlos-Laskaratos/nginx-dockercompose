# nginx - dockercompose
A docker-compose project with two containers one being [nginx](https://nginx.org/) reverse proxy and the other being a container with the [certbot](https://certbot.eff.org/) service offering certificates issuance in an isolated environment.

# Getting Started
Bellow is the process of deploying this set up.

## Installing and Setting up Docker Rootless
This configuration is meant to be deployed with docker rootless for increased security. There is no reason why it shoudn't work with docker rootfull but it hasn't been tried in that context.

Before installing docker rootless create a new user dedicated to running the docker services.

```
sudo useradd -s /bin/bash dockeruser   
sudo passwd -l dockeruser
```

To install docker rootless follow the official instructions [here](https://docs.docker.com/engine/security/rootless/).

Be carefull to login with the `dockeruser` when setting up rootless. For users that cannot be logged in there are instructions [here](https://docs.docker.com/engine/security/rootless/troubleshoot/#unable-to-install-with-systemd-when-systemd-is-present-on-the-system).

Additionally it is necessary to allow unprivileged ports to start at 0 for nginx to work properly. There are instructions for that in the official documentation as well [here](https://docs.docker.com/engine/security/rootless/tips/#exposing-privileged-ports).


## Clone and Make Directories
Create the root directory of the service and give proper permissions. 

```bash
mkdir /srv/docker
chown dockeruser:admins docker
# Admins is a group of user that have admin privileges in my server that my admin user admin is a part of. I set this group so that I can easily read and write without having to envoke sudo or login to dockeruser.
chmod 770 docker 
```

Clone the repository.
``` bash
cd /srv/docker
git clone https://github.com/Pavlos-Laskaratos/nginx-dockercompose.git
```

You should now see the `docker-compose.yaml` and this file in your local machine. Before starting the containers the following directories must be created.
```bash
mkdir /srv/configs/nginx
mkdir /srv/data/nginx/conf.d
mkdir /srv/data/nginx/sites-enabled
mkdir /srv/data/nginx/sites-available
mkdir /srv/data/nginx/ssl
chown -R dockeruser:admin /srv/data/nginx /srv/configs/nginx
chmod -R 770 /srv/data/nginx /srv/configs/nginx
```

The default `nginx.conf` must be manually created because otherwise it would be left empty without the nginx process being able to initialiaze it since the `config` volume will be read only immediately.

```bash
cd /srv/configs/nginx
sudo -u dockeruser docker run --rm nginx:latest cat /etc/nginx/nginx.conf | sudo tee nginx.conf > /dev/null           
```

## Run Containers
At this moment we can compose up as the `dockeruser`

```bash
cd /srv/docker/nginx
sudo -u dockeruser docker compose up -d
```

- Snippets
- Certbot commands
- Set up cron


# Certbot Certificates
To obtain a certificates the bellow snippet must be included inside the server block of the nginx configuration of the domain we want to obtain a certificate for.
```example.example.com.conf
server {
	listen 80;
	server_name example.example.com;
	location ^~ /.well-known/acme-challenge/ {
    	root /var/www/html;
    	default_type "text/plain"; 
	}
}
```


## Obtain certificate
I prefer getting certificates manually by executing this command:
```bash
sudo -u dockeruser docker exec -it reverseproxy-certbot certbot certonly --webroot -w /var/www/html -d example.example.com # --dry-run   
```

## Renew Certificates
Renewal is handled automatically by certbot via the following command.
```bash
sudo -u dockeruser docker exec -it reverseproxy-certbot certbot renew
```

It is the responsibility of the user to periodically renew the certificates and to reload nginx with the possible newer certificates.

# Troubleshooting
Sometimes a get a problem with certificate issuance. This usually resolves by manually creating the acme-challenge directory inside the docker volume.

```
sudo -u dockeruser docker exec -it <nginx-container> bash
mkdir -p /var/www/html/.well-known/acme-challenge
```

