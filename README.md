# Odoo Installation on ubuntu 20.04
<!-- https://github.com/odoo/odoo -->
<!-- https://www.odoo.com/documentation/14.0/administration/install.html -->

```
sudo apt update && sudo apt upgrade -y

sudo apt install -y python3-pip

sudo apt install postgresql postgresql-client -y
sudo -u postgres createuser -s $USER
createdb $USER

sudo apt install -y python3-dev libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev \
    libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev libfreetype6-dev \
    liblcms2-dev libwebp-dev libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev

cd /tmp
wget https://github.com/odoo/odoo/archive/refs/heads/14.0.zip

sudo apt install unzip -y

unzip 14.0.zip -d ~/odoo

cd ~/odoo/odoo-14.0/

pip3 install setuptools wheel
pip3 install -r requirements.txt

python3 odoo-bin --addons-path=addons -d mysupererpcrm </dev/null &>/dev/null &


sudo apt install nginx -y

sudo snap install core; sudo snap refresh core

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx


upstream odoo {
  server 127.0.0.1:8069;
}
upstream odoochat {
  server 127.0.0.1:8072;
}

# http -> https
server {
  listen 80;
  server_name odoo.mycompany.com;
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443;
  server_name odoo.mycompany.com;
  proxy_read_timeout 720s;
  proxy_connect_timeout 720s;
  proxy_send_timeout 720s;

  # Add Headers for odoo proxy mode
  proxy_set_header X-Forwarded-Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_set_header X-Real-IP $remote_addr;

  # SSL parameters
  ssl on;
  ssl_certificate /etc/ssl/nginx/server.crt;
  ssl_certificate_key /etc/ssl/nginx/server.key;
  ssl_session_timeout 30m;
  ssl_protocols TLSv1.2;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;

  # log
  access_log /var/log/nginx/odoo.access.log;
  error_log /var/log/nginx/odoo.error.log;

  # Redirect longpoll requests to odoo longpolling port
  location /longpolling {
    proxy_pass http://odoochat;
  }

  # Redirect requests to odoo backend server
  location / {
    proxy_redirect off;
    proxy_pass http://odoo;
  }

  # common gzip
  gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
  gzip on;
}
```