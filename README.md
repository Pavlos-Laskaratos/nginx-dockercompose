```
sudo useradd -s /bin/bash dockerservice    
sudo passwd -l dockerservice     
```

```
sudo setcap cap_net_bind_service=ep $(which rootlesskit)                                      │
│systemctl --user restart docker       
```


## Dont forget
```
mkdir /srv/configs/reverserproxy
mkdir /srv/data/reverserproxy
chown docker_user:docker_user configs
chown docker_user:docker_user data
touch .../nginx.config
mkdir .../reverseproxy/conf.d
```

```
docker run --rm nginx:latest cat /etc/nginx/nginx.conf > nginx.conf
```


# Certbot Commands
## Obtain certificate
```
certbot certonly --webroot -w /var/www/html -d $host_name --dry-run   
```

## Renew Certificates
```
│/usr/bin/certbot renew    
```

