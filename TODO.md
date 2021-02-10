# TODO

- setup email server
	- https://mailinabox.email/
- export mailing list data from google groups
- import list data to discourse as topic
- enable create-topic-by-email
	- https://meta.discourse.org/t/replacing-mailing-lists-email-in/13099

## LOG

### 2021-02-09

Creating Discourse user

	`adduser discourse`

Adding discourse to sudoers

	`usermod -a -G sudo discourse`

Add `docker` group
	
	`groupadd docker`

Add discourse to docker group

	`usermod -a -G docker discourse`

Install Docker

	`apt-get install docker docker-compose`

Following Discourse "Cloud" install via 

	- https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md



### 2021-01-10

Linode VPS -- 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS


Added A Record in domain DNS to point to server ip : 198.74.51.225

	`A  forum.shiny.ooo  198.74.51.225  1798`


Check to see if DNS is resolving to ip

```
	$ getent hosts forum.shiny.ooo
	# 198.74.51.225   forum.shiny.ooo
```


Created local (bare) git repo

	`/srv/git/repos/setup.git`


Installed some stuff

```
	$ whoami
	# root
	$ apt update
	$ apt upgrade
	$ apt install tmux
```
