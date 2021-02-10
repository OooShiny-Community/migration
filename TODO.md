# TODO

- [ ] Setup email server
	- https://mailinabox.email
	- https://mailcow.email
- [ ] Export mailing list data from Google groups
- [ ] Import list data to discourse as topic
- [ ] Enable create-topic-by-email
	- https://meta.discourse.org/t/replacing-mailing-lists-email-in/13099
- [x] Install Discourse via Docker
- [x] Create `discourse` user and `docker` group
- [x] Setup/Resolve DNS
- [x] Setup VPS


## LOG


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



