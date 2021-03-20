# Discourse Setup

## Server Setup

ip: 198.74.51.225

Linode VPS, 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS

## Prerequesits

Set up a mailserver.


## Adding DNS Record

Added A Record in domain DNS to point to server ip : 198.74.51.225

    A  forum.shiny.ooo  50.116.8.231  1798


## Environment

    whoami
    # root
    apt update
    apt upgrade
    apt-get install docker docker-compose

Check to see if DNS is resolving to ip

    getent hosts forum.shiny.ooo
    # 50.116.8.231   forum.shiny.ooo

Creating Discourse user and add to sudoers

    adduser discourse
    usermod -aG sudo discourse

Add `docker` group

    groupadd docker

Add discourse to docker group

    usermod -aG docker discourse



## Install Discourse via Docker

Referencing Discourse "Cloud" docker install via

- https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md

With some changes, as we are installing Discourse on the same server as our mail server.

As root, Clone the Official Discourse Docker Image into /var/discourse. You must be root throughout the install process

    sudo -s
    git clone https://github.com/discourse/discourse_docker.git /var/discourse
    cd /var/discourse

Edit samples/standalone.yml

    ...

    templates:
      - "templates/postgres.template.yml"
      - "templates/redis.template.yml"
      - "templates/web.template.yml"
      - "templates/web.ratelimited.template.yml"
      - "templates/web.socketed.template.yml"
    ...

    # comment out exposed ports (we will handle this with nginx outside the container)
    #  - "80:80"   # http
    #  - "443:443" # https

    ...

    DISCOURSE_HOSTNAME: 'forum.shiny.ooo'
    DISCOURSE_DEVELOPER_EMAILS: 'admin@shiny.ooo'
    DISCOURSE_SMTP_ADDRESS: 'mail.shiny.ooo'
    DISCOURSE_SMTP_PORT: '587'
    DISCOURSE_SMTP_USER_NAME: 'admin@shiny.ooo'
    DISCOURSE_SMTP_PASSWORD: '<password>'
    DISCOURSE_NOTIFICATION_EMAIL: 'forum@shiny.ooo'

    ...


Build (first time)

    ./launcher bootstrap app

Rebuild (if variables are edited after building)

    ./launcher rebuild app

Start App

    ./launcher start app


## Generate SSL certs for forum subdomain (forum.shiny.ooo) as additional domain

    sudo certbot certonly --webroot --agree-tos -d mail.shiny.ooo,mail.oooshiny.email,forum.shiny.ooo --cert-name shiny.ooo --email admin@shiny.ooo -w /var/www/html


## Create a reverse proxy for Discourse in Nginx

Create file `/etc/nginx/sites-available/20-forum.shiny.ooo.conf`

    # HTTP
    server {
        # Listen on ipv4
        listen 80;
        listen [::]:80;

        server_name forum.shiny.ooo;

        # Redirect all insecure http:// requests to https://
        return 301 https://$host$request_uri;
    }

    # HTTPS
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name forum.shiny.ooo;

        root /var/www/html;

        include /etc/nginx/templates/misc.tmpl;
        include /etc/nginx/templates/ssl.tmpl;
        include /etc/nginx/templates/discourse.tmpl;
    }

Link sites file in `sites-enabled/`

    cd /etc/nginx/sites-enabled
    ln -s ../sites-available/20-forum.shiny.ooo.conf

Test and Reload nginx

    nginx -t
    systemctl restart nginx
    systemctl status nginx


## Set up Discourse via Browser

Navigate to `forum.shiny.ooo` to complete setting up Discourse


## Run Discourse-Doctor for testing install and email settings

    cd /var/discourse
    sudo ./discourse-doctor


## Post Install

Enable automatic upgrades

    dpkg-reconfigure -plow unattended-upgrades


## Links

- [Discourse Cloud Install](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md)
- [Troubleshooting email on a new Discourse install](https://meta.discourse.org/t/troubleshooting-email-on-a-new-discourse-install/16326)
- [Configure Reply-By-Mail](https://meta.discourse.org/t/set-up-reply-via-email-support/14003)
- [Configure Backups](https://meta.discourse.org/t/configure-automatic-backups-for-discourse/14855)
- [Running Discourse alongside other websites](https://meta.discourse.org/t/running-other-websites-on-the-same-machine-as-discourse/17247)
- [Multiple Discourse Installs](https://linuxhandbook.com/multiple-discourse-install/)
- [Add offline page when rebuilding](https://meta.discourse.org/t/adding-an-offline-page-when-rebuilding/45238)