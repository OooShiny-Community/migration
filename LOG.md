# LOG

### 2021-02-19

Adding `git` group

    groupadd git

Adding `admin` user
	
	adduser admin
	usermod -a -G sudo admin
	usermod -a -G docker admin
    usermod -a -G git admin

Switching to Admin user

Installing ZSH shell

    sudo apt-get install zsh zplug

Installing dotfiles

    https://github.com/quilime/cfg

Experimenting with SMTP Server

Making a few folders

    [admin@shiny] ~/dockermail $ sudo mkdir -p /opt/dockermail/mail                                                                                                                                                                                                                                   ±[master]
    [admin@shiny] ~/dockermail $ sudo mkdir -p /opt/dockermail/settings


### 2021-02-11


Setting up Cloudflare for DNS

Adding `templates/cloudflare.template.yml` to app.yml and rebuilding

via: https://meta.discourse.org/t/cloudflare-with-discourse/137456/7


### 2021-02-10

Creating new VPS for Mailserver

	Linode VPS, 1 GB Ram, 1 CPU, Ubuntu 18.04.5 LTS

Using 18.04 as per the mailinabox docs



### 2021-02-09

Creating Discourse user

	adduser discourse

Adding discourse to sudoers

	usermod -a -G sudo discourse

Add `docker` group
	
	groupadd docker

Add discourse to docker group

	usermod -a -G docker discourse

Install Docker

	apt-get install docker docker-compose

Following Discourse "Cloud" install via 

- https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md



### 2021-01-10

Added A Record in domain DNS to point to server ip : 198.74.51.225

	A  forum.shiny.ooo  198.74.51.225  1798


Check to see if DNS is resolving to ip

	$ getent hosts forum.shiny.ooo
	# 198.74.51.225   forum.shiny.ooo


Created local (bare) git repo

	/srv/git/repos/setup.git


Installed some stuff

	whoami
	# root
	apt update
	apt upgrade
	apt install vim tmux


Creating VPS

	Linode VPS, 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS



