# Discourse Setup

## Server Setup

ip: 198.74.51.225

Linode VPS, 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS


## Adding DNS Record

Added A Record in domain DNS to point to server ip : 198.74.51.225

    A  forum.shiny.ooo  198.74.51.225  1798


# Initial Setup

    whoami
    # root
    apt update
    apt upgrade

Check to see if DNS is resolving to ip

    getent hosts forum.shiny.ooo
    # 198.74.51.225   forum.shiny.ooo

Creating Discourse user and add to sudoers

    adduser discourse
    adduser discourse sudo

Add `docker` group

    groupadd docker

Add discourse to docker group

    usermod -a -G docker discourse

Install Docker

    apt-get install docker docker-compose


## Install Discourse via Docker

Following Discourse "Cloud" docker install via

- https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md

Setting up Cloudflare for DNS

Adding `templates/cloudflare.template.yml` to app.yml and rebuilding

via: https://meta.discourse.org/t/cloudflare-with-discourse/137456/7


## Mail server settings in /var/discourse/containers/app.yml

    DISCOURSE_DEVELOPER_EMAILS: admin@shiny.ooo
    DISCOURSE_SMTP_ADDRESS: mail.shiny.ooo
    DISCOURSE_SMTP_PORT: 587
    DISCOURSE_SMTP_USER_NAME: admin@shiny.ooo
    DISCOURSE_SMTP_PASSWORD: <password>


## Post Install

Install fail2ban and netdata

    apt install fail2ban

Install [netdata](https://learn.netdata.cloud/docs/agent/packaging/installer#automatic-one-line-installation-script)

    bash <(curl -Ss https://my-netdata.io/kickstart.sh)

View server stats. More [config](https://learn.netdata.cloud/docs/quickstart/single-node)

    # TODO Enforce SSL and login
    http://198.74.51.225:19999


## Links

- [Discourse Cloud Install](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md)
- [Troubleshooting email on a new Discourse install](https://meta.discourse.org/t/troubleshooting-email-on-a-new-discourse-install/16326)
- [Configure Reply-By-Mail](https://meta.discourse.org/t/set-up-reply-via-email-support/14003)
- [Configure Backups](https://meta.discourse.org/t/configure-automatic-backups-for-discourse/14855)