# TODO

- [ ] Setup email server
	- https://mailinabox.email
	- https://github.com/tonioo/modoboa
	- https://github.com/sovereign/sovereign
	- https://mailcow.email
	- https://github.com/mailcow/mailcow-dockerized
	- https://mailu.io/
	- https://www.iredmail.org/
	- https://workaround.org/ispmail/buster/
	- https://sealedabstract.com/code/nsa-proof-your-e-mail-in-2-hours/
	- https://workaround.org/ispmail
	- https://github.com/jeboehm/docker-mailserver
	- https://github.com/namshi/docker-smtp
	- https://github.com/docker-mailserver/docker-mailserver

- [ ] Conform outgoing mail
- [ ] Setup DNS
	- cloudflare
- [ ] Setup SSL/HTTPS
	- LetsEncrypt
- [ ] Setup receiving incoming mail to create new topics
	- https://meta.discourse.org/t/discourse-vs-email-mailing-lists/54298/4
	- https://meta.discourse.org/t/start-a-new-topic-via-email/62977	
	- https://github.com/discourse/mail-receiver
	- https://meta.discourse.org/t/configuring-reply-via-email/42026
	- https://meta.discourse.org/t/straightforward-direct-delivery-incoming-mail/49487
	- https://meta.discourse.org/t/filtering-known-bad-sender-domains-from-your-mail-receiver/118760
	- 
- [ ] Export mailing list data from Google groups
- [ ] Import list data to discourse as topic
- [ ] Enable create-topic-by-email
	- https://meta.discourse.org/t/replacing-mailing-lists-email-in/13099
- [x] Install Discourse via Docker
- [x] Create `discourse` user and `docker` group
- [x] Setup/Resolve DNS
- [x] Setup VPS


## LOG

### 2021-02-19

Adding `admin` user
	
	adduser admin
	usermod -a -G sudo admin
	usermod -a -G docker admin

Switching to Admin user

Installing ZSH shell

    sudo apt-get install zsh zplug

Installing dotfiles

    https://github.com/quilime/cfg



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



