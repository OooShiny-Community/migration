# Mailserver and Discourse Installation

This is documentation on how to install a Mailserver and Discourse along-side eachother on the same VPS. This is not recommended by the officla Discourse install due to complexities, so this guide is an attempt to document the process.

- Email: [iRedMail](https://www.iredmail.org/)
- Forum Software: [Discourse](https://www.discourse.org/)

## Server

ip: 50.116.8.231

VPS: Linode VPS, 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS

Relevant domains and hostnames:

- mail.shiny.ooo - Server hostname and mailserver
- mail.oooshiny.email - Secondary mailserver (.ooo tld has been said to be a "dirty" tld for sending email)
- forum.shiny.ooo - Forum software


## Create DNS MX Record(s)

    DNX Record Type     Name    Value                TTL
    MX                  @       mail.shiny.ooo       90sec


## Preparing Mail Server Installation

OS upgrades

    sudo -s # switch to root
    apt update
    apt upgrade -y

Creating mailadmin user

    adduser mailadmin
    usermod -aG sudo mailadmin
    su - mailadmin

Set a fully qualified domain name (FQDN) for the server

    sudo hostnamectl set-hostname mail.shiny.ooo

Edit `/etc/hosts`, add the following

    127.0.0.1       mail.shiny.ooo localhost

Verify the changes

    hostname -f


## Install iRedMail

[iRedMail](https://www.iredmail.org/) is a shell script that automatically installs and configures all necessary mail server components.

This server setup was inspired and modifed from this tutorial: [How to Easily Set up a Full-Fledged Mail Server on Ubuntu 20.04 with iRedMail](https://www.linuxbabe.com/mail-server/ubuntu-20-04-iredmail-server-installation)

Download iRedMail

    wget https://github.com/iredmail/iRedMail/archive/1.3.2.tar.gz

Extract the archived file.

    tar xvf 1.3.2.tar.gz

Then cd into the newly-created directory.

    cd iRedMail-1.3.2/

Add executable permission to the `iRedMail.sh` script.

    chmod +x iRedMail.sh

Run the install script with sudo privilege.

    sudo bash iRedMail.sh

The mail server setup wizard will start. Use the `Tab` to select Yes and press `Enter`.

Use default storage path: `/var/vmail`

Use `Nginx` for Webserver

Use `PostGres` for Backend

Enter in mail domain name

    mail.shiny.ooo

Set password for mail domain admin

Chose optional components

    [*] RoundCubeMail
    [ ] SOHo <- optional groupware
    [*] netdata
    [*] iRedAdmin
    [*] Fail2Ban

Review the config. Press `Y` to begin the installation.

At the end of installation, choose y to use the firewall rules provided by iRedMail and restart the firewall.

When the installation is complete. You will be notified the URL of webmail, web admin panel and the login credentials. The iRedMail.tips file contains additional info about the server

     /home/mailadmin/iRedMail-1.3.2/iRedMail.tips

Config

    /home/mailadmin/iRedMail-1.3.2/config

Reboot the server

    sudo shutdown -r now

Once rebooted, visit the web panel

    https://mail.shiny.ooo/iredadmin/


## Install Let's Encrypt TLS Cert

Install CertBot

    sudo apt install certbot

Generate certificate

    sudo certbot certonly --webroot --agree-tos --email admin@shiny.ooo -d shiny.ooo -w /var/www/html/

When it asks if you want to receive communications from EFF, choose `No`.

The certificate and chain should be saved at `/etc/letsencrypt/live/shiny.ooo/`

Edit Nginx ssl template `/etc/nginx/templates/ssl.tmpl`

Find the following 2 lines.

    ssl_certificate /etc/ssl/certs/iRedMail.crt;
    ssl_certificate_key /etc/ssl/private/iRedMail.key;

Replace them with:

    ssl_certificate /etc/letsencrypt/live/shiny.ooo/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/shiny.ooo/privkey.pem;

Save and close the file. Test nginx configuration and reload.

    sudo nginx -t
    sudo systemctl reload nginx


## Installing TLS Certificate in Postfix and Dovecot

Set up Postfix SMTP server and Dovecot IMAP server to use the Let’s Encrypt issued certificate.

Edit the main configuration file of Postfix `/etc/postfix/main.cf`

Find the following 3 lines. (line 95, 96, 97).

    smtpd_tls_key_file = /etc/ssl/private/iRedMail.key
    smtpd_tls_cert_file = /etc/ssl/certs/iRedMail.crt
    smtpd_tls_CAfile = /etc/ssl/certs/iRedMail.crt

Replace them with:

    smtpd_tls_key_file = /etc/letsencrypt/live/shiny.ooo/privkey.pem
    smtpd_tls_cert_file = /etc/letsencrypt/live/shiny.ooo/cert.pem
    smtpd_tls_CAfile = /etc/letsencrypt/live/shiny.ooo/chain.pem

Save and close the file. Reload Postfix.

    sudo systemctl reload postfix

Edit the main configuration file of Dovecot `/etc/dovecot/dovecot.conf`

Find the following 2 lines. (line 47, 48)

    ssl_cert = </etc/ssl/certs/iRedMail.crt
    ssl_key = </etc/ssl/private/iRedMail.key

Replace them with:

    ssl_cert = </etc/letsencrypt/live/shiny.ooo/fullchain.pem
    ssl_key = </etc/letsencrypt/live/shiny.ooo/privkey.pem

Save and close the file. Reload dovecot.

    sudo systemctl reload dovecot


## Send Test Email

Log into the iredadmin panel with the postmaster mail account postmaster@shiny.ooo. In the Add tab, you can add additional domains or email addresses.

After you create an address, you can visit the Roundcube webmail address and login with the new mail user account.

    https://mail.shiny.ooo/mail/

Test sending and receiving. For troubelshooting, [Errors you may see while maintaining iRedMail server](https://docs.iredmail.org/errors.html#recipient-address-rejected-sender-is-not-same-as-smtp-authenticate-username)


## Fail2ban Allowlist

Add the IP address of any connecting servers to the allowlist by editing the jail.local file `/etc/fail2ban/jail.local`

Add the server IP address to the ignore list

    ignoreip = 127.0.0.1 127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 50.116.8.231/24

Save and close the file. Then restart Fail2ban.

    sudo systemctl restart fail2ban


## Improving Email Deliverablity

To prevent your emails from being flagged as spam, set PTR, SPF, DKIM and DMARC records.


### PTR Record

To check the PTR record for an IP address, run:

    dig -x 50.116.8.231 +short

or

    host 50.116.8.231


### SPF Record

SPF (Sender Policy Framework) record specifies which hosts or IP address are allowed to send emails on behalf of a domain.

    DNX Record Type     Name    Value           TTL
    TXT                 @       v=spf1 ms ~all  90sec

Explanation:

- `TXT` indicates this is a TXT record.
- `@` in the name field represents the main domain name.
- `v=spf1` indicates this is a SPF record and the version is SPF1.
- `mx` means all hosts listed in the MX records are allowed to send emails for your domain and all other hosts are disallowed.
- `~all` indicates that emails from your domain should only come from hosts specified in the SPF record. Emails that are from other hosts will be flagged as forged.

Check if your SPF record is propagated to the public Internet, you can use the dig utility on your Linux machine like below:

    dig mail.shiny.ooo txt


### DKIM Record

DKIM (DomainKeys Identified Mail) uses a private key to digitally sign emails sent from your domain. Receiving SMTP servers verify the signature by using the public key, which is published in the DNS DKIM record.

The iRedMail script automatically configured DKIM for your server. Run the following command to show the DKIM public key.

    sudo amavisd-new showkeys

The DKIM public key is in the parentheses in the response.

In your DNS manager, create a TXT record with `dkim._domainkey` in the name field. Copy everything in the parentheses and paste into the value field. Delete all double quotes and line breaks.

    DNX Record Type     Name                Value                   TTL
    TXT                 dkim._domainkey     v=DKIM1l; p=MIIBIj...   90sec

After saving your changes, run the following to test if your DKIM record is correct.

    sudo amavisd-new testkeys

Server DKIM Records:

mail.shiny.ooo

    v=DKIM1; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArBoREgtvoffMW5RB4IaI5c0P1flPaJL3+tp+LQl6NKaBVHZGMpLqbbB4khoclBU7vscDDbgrqdcwFYKAJcn1XxxLM7zl2G5X4yt5nZrqz0b/AaBsgyVirGVZWziX0pKnx6PY8ipDFzZ5oCKLAQKJmsuQ1Gy2Dpaw5pWztBXLY9khyeHiaASiV8UsJW7CAfehqJFWzGnw+0xlwDo6//tYIrYuN0YGIEnBuFQ61OoGg3OAmMbvoCYZafu2nyoQkY+LDNW/0X0IQ1zTtynv5FRJosBxBh5lyFhQ/twwBWpS0NpbT0llh6EJ46YJY8t2YfN8VfV4Q4llLH1B0T034AtAAQIDAQAB

    v=DMARC1; p=none; pct=100; rua=mailto:dmarc@shiny.ooo

mail.oooshiny.email

    v=DKIM1; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz+xV07vNHVqE2QsfcS6E51CqlgnpIQfjjTCHhLXLxhXpDtJH4VzNfsisZo1euDNg6dWwiA32hr+jnjTIiieuCAdbvlp3ZmhoK7UjzH3WcmBaZk8n9q9EP840FtjO1uXs/HnAGahm373btkEGl18sjZJdUAxFk+w1tH7kwjYKwdp36D1Q2VAhQjntWFcIgSKHq9KaMT5suAya/10igI9JCgnB5lAjJIDrKYyEnkPRLLXBR0ONzQ6919aOtk3WGp9AMlF9b20Mgr4of1qGL3sQP5mAYgqn3yMpfUsQk4KjwZ//fr1bhSJe6Z4wlOhdnbA7CCu5heCzG5Gb2MxVCwv64wIDAQAB

    v=DMARC1; p=none; pct=100; rua=mailto:dmarc@oooshiny.email

### DMARC Record

DMARC (Domain-based Message Authentication, Reporting and Conformance) can help receiving email servers to identify legitimate emails and prevent your domain name from being used by email spoofing.

To create a DMARC record, go to your DNS manager and add a TXT record. In the name field, enter `_dmarc`. In the value field, enter the following. (Be sure to create the dmarc@shiny.ooo email address).

    v=DMARC1; p=none; pct=100; rua=mailto:dmarc@shiny.ooo

    DNX Record Type     Name       Value                                                    TTL
    TXT                 _dmarc     v=DMARC1; p=none; pct=100; rua=mailto:dmarc@shiny.ooo    5min


### Auto Renew TLS Cert

Let’s Encrypt issued TLS certificate is valid for 90 days. We use Cron to automatically renew the certificate.

    sudo certbot renew -w /var/www/html/

Use the `--dry-run` option to test the renewal process, instead of doing a real renewal.

    sudo certbot renew -w /var/www/html/ --dry-run

Edit the SSL virtual host `/etc/nginx/sites-enabled/00-default-ssl.conf`. Add the following lines.

    location ~ /.well-known/acme-challenge {
        root /var/www/html/;
        allow all;
    }

Save and close the file. Test Nginx config and reload.

    sudo nginx -t
    sudo systemctl reload nginx


### Create Cron Job

If the dry run is successful, create a Cron job to automatically renew the cert. Open root user’s crontab file:

    sudo crontab -e

Add the following line at the bottom of the file.

    @daily certbot renew -w /var/www/html/ --quiet && systemctl reload postfix dovecot nginx


### Adding Swap Space

Various components use a fair amount of RAM. Ff there’s not enough RAM on the server,mail components will hang, preventing outgoing mails, and other issues. Add swap space with 4G capacity in root file system:

    sudo fallocate -l 4G /swapfile

Only root can read and write to it

    sudo chmod 600 /swapfile

Format it to swap:

    sudo mkswap /swapfile
    # Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
    # no label, UUID=0aab5886-4dfb-40d4-920d-fb1115c67433

Enable the swap file

    sudo swapon /swapfile

mount the swap space at system boot time, edit `/etc/fstab`

Add the following line at the bottom

    /swapfile    swap    swap     defaults    0   0

Save and close the file. Then reload systemd and clamav

    sudo systemctl daemon-reload
    sudo systemctl restart clamav-daemon

### Adding Additional Domains (Optional)

#### Add a new mail domain and user in iRedMail admin panel.

#### Creating MX, A and SPF record for the new mail domain

In your DNS manager, add MX record for the new domain like below.

    Record Type    Name      Value
    MX             @         mail.oooshiny.email

The A record points to your mail server’s IP address.

    Record Type    Name     Value
    A              mail     50.116.8.231

If your server uses IPv6 address, be sure to add AAAA record.

Then create SPF record to allow the MX host to send email for the new mail domain.

    Record Type    Name      Value
    TXT            @         v=spf1 mx ~all


#### Setting up DKIM signing for the new domain

You need to tell amavisd to sign every outgoing email for the new mail domain. Edit `/etc/amavis/conf.d/50-user`

Find the following line,

    dkim_key('mail.shiny.ooo', 'dkim', '/var/lib/dkim/mail.shiny.ooo.pem');

Add another line to specify the location of the private key of the second domain.

    dkim_key('mail.oooshiny.email', 'dkim', '/var/lib/dkim/mail.oooshiny.email.pem');

In @dkim_signature_options_bysender_maps section, add the following line.

    "mail.oooshiny.email" => { d => "mail.oooshiny.email", a => 'rsa-sha256', ttl => 10*24*3600 },

Save and close the file. Then generate the private key for the second domain.

    sudo amavisd-new genrsa /var/lib/dkim/mail.oooshiny.email.com.pem 2048

Restart Amavis.

    sudo systemctl restart amavis

Display the public keys.

    sudo amavisd-new showkeys

All public keys will be displayed. We need the public key of the second domain, which is in the parentheses.

    amavis show keys

In your DNS manager, create a TXT record for the second domain. Enter `dkim._domainkey` in the Name field. Copy everything in the parentheses and paste into the value field. Delete all double quotes. (You can paste it into a text editor first, delete all double quotes, the copy it to your DNS manager. Your DNS manager may require you to delete other invalid characters, such as carriage return.)

After saving your changes. Check the TXT record with this command.

    dig TXT dkim._domainkey.mail.oooshiny.email

Now you can run the following command to test if your DKIM DNS record is correct.

    sudo amavisd-new testkeys

If the DNS record is correct, the test will pass.

    # TESTING#1 mail.shiny.ooo: dkim._domainkey.mail.shiny.ooo => pass
    # TESTING#2 mail.oooshiny.email: dkim._domainkey.mail.oooshiny.email => pass

Note that your DKIM record may need sometime to propagate to the Internet. Depending on the domain registrar you use, your DNS record might be propagated instantly, or it might take up to 24 hours to propagate. Use [DKIM Check](https://www.dmarcanalyzer.com/dkim/dkim-check/), enter dkim as the selector and enter your domain name to check DKIM record propagation.


#### Setting Up DMARC Record For the New Domain

To create a DMARC record, go to your DNS manager and add a TXT record. In the name field, enter `_dmarc`. In the value field, enter the following:

    v=DMARC1; p=none; pct=100; rua=mailto:dmarc@oooshiny.email

The above DMARC record is a safe starting point. To see the full explanation of DMARC, please read [Creating DMARC Record to Protect Your Domain Name From Email Spoofing])https://www.linuxbabe.com/mail-server/create-dmarc-record_


#### Setting up RoundCube, Postfix and Dovecot for Multiple Domains

It makes sense to let users of the first domain use mail.shiny.ooo and users of the second domain use mail.oooshiny.email when using RoundCube webmail.

    cd /etc/nginx/

Create a blank server block file for the second domain in /etc/nginx/sites-enabled/ directory.

    sudo touch sites-enabled/10-mail.oooshiny.email.conf

Copy the default HTTP site configurations to the file.

    cat sites-enabled/00-default.conf | sudo tee -a sites-enabled/10-mail.oooshiny.email.conf

Copy the default SSL site configurations to the file.

    cat sites-enabled/00-default-ssl.conf | sudo tee -a sites-enabled/10-mail.oooshiny.email.conf

Edit the virtual host file `sites-enabled/10-mail.oooshiny.email.conf`

Find the following line.

    server_name _;

We need to change the server_name to mail.oooshiny.email, because later we need to use Certbot to generate a new tls certificate.

    server_name mail.oooshiny.email;

There are 2 instances of server_name, you need to change both of them. Save and close the file. Then test Nginx configuration.

    sudo nginx -t

If the test is successful, reload Nginx for the changes to take effect.

    sudo systemctl reload nginx

Now use Certbot webroot plugin to obtain TLS certificate for all your mail domains, so you will have a single TLS certificate with multiple domain names on it.

    sudo certbot certonly --webroot --agree-tos -d mail.shiny.ooo,mail.oooshiny.email --cert-name shiny.ooo --email admin@shiny.ooo -w /var/www/html

Notice that in the above command, we specified the cert name using the root domain, which will be used in the file path, so you don’t have to change the file path in Postfix or Dovecot configuration file.

When it asks if you want to update the existing certificate to include the new domain, answer U and hit Enter. Now you should see a message which indicates the multi-domain certificate is successfully obtained. Reload Nginx to pick up the new certificate.

    sudo systemctl reload nginx # reload
    sudo systemctl status nginx # check status

Reload Postfix SMTP server and Dovecot IMAP server in order to let them pick up the new certificate.

    sudo systemctl reload postfix dovecot

You should now be able to use different domains to access RoundCube webmail.

#### SPF and DKIM Check

Now you can use your desktop email client or webmail client to send a test email to check-auth@verifier.port25.com and get a free email authentication report. You can also test email score at https://www.mail-tester.com and also test email placement with GlockApps. If DKIM check fails, you can go to https://www.dmarcanalyzer.com/dkim/dkim-check/ to see if there are any errors with your DKIM record.


## Logs Location

https://docs.iredmail.org/file.locations.html

    sudo netstat -tulpn # show ports
    /var/log/nginx/ # nginx logs location





# Installing Discourse


## Adding DNS Record

Added A Record for a new subdomain to point to the server ip

    A  forum.shiny.ooo  50.116.8.231


## Environment

Confirm env is up to date

    sudo -s # switch to root
    apt update
    apt upgrade
    apt-get install docker docker-compose

Check to see if DNS is resolving to ip

    getent hosts forum.shiny.ooo
    # 50.116.8.231   forum.shiny.ooo


## Install Discourse via Docker

Referencing Discourse "Cloud" docker install via

- https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md

With some changes, as we are installing Discourse on the same server as our mail server.

As root, Clone the Official Discourse Docker Image into /var/discourse. You must be root throughout the install process

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

    certbot certonly --webroot --agree-tos -d mail.shiny.ooo,mail.oooshiny.email,forum.shiny.ooo --cert-name shiny.ooo --email admin@shiny.ooo -w /var/www/html


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
    ./discourse-doctor


## Post Install

Enable automatic upgrades

    dpkg-reconfigure -plow unattended-upgrades


## More info and Links

- [How to Easily Set up a Full-Fledged Mail Server on Ubuntu 20.04 with iRedMail](https://www.linuxbabe.com/mail-server/ubuntu-20-04-iredmail-server-installation)
- [Errors you may see while maintaining iRedMail server](https://docs.iredmail.org/errors.html#recipient-address-rejected-sender-is-not-same-as-smtp-authenticate-username)
- [How to Host Multiple Mail Domains in iRedMail with Nginx](https://www.linux`.com/mail-server/set-up-iredmail-multiple-domains-nginx)
- [How to Set up a Backup Email Server with Postfix on Ubuntu (Complete Guide)](https://www.linuxbabe.com/mail-server/how-to-set-up-a-backup-email-server-postfix-ubuntu)
- [Set Up OpenDMARC with Postfix on Ubuntu to Block Email Spoofing/Spam](https://www.linuxbabe.com/mail-server/opendmarc-postfix-ubuntu)
- [Block Email Spam By Checking Header and Body in Postfix/SpamAssassin](https://www.linuxbabe.com/mail-server/block-email-spam-check-header-body-with-postfix-spamassassin)
- [How to set up SMTP relay between 2 Postfix SMTP servers on Ubuntu](https://www.linuxbabe.com/mail-server/smtp-relay-between-2-postfix-smtp-servers)
- [Secure Email Communication in iRedMail](http://doc.samplezone.ch/iredmail/version-0-9-2/ssl-tls-secure-communication/tls-ssl-starttls-iredmail/)
- [Add IP to Allowlist for Fail2Ban](https://linuxhandbook.com/fail2ban-basic/#how-to-unban-ip-blocked-by-fail2ban)
- [List all Banned Addresses Gist](https://gist.github.com/kamermans/1076290)
- [Disable Spam Filtering for Outgoing](https://docs.iredmail.org/disable.spam.virus.scanning.for.outgoing.mails.html)
- [Completely disable Amavisd + ClamAV + SpamAssassin](https://docs.iredmail.org/completely.disable.amavisd.clamav.spamassassin.html)
- [Unblock IP's that have been blocked by fail2ban](https://linuxhandbook.com/fail2ban-basic/#how-to-unban-ip-blocked-by-fail2ban)
- [Show Ignored IP's Gist](https://gist.github.com/kamermans/1076290)


### Discourse

- [Discourse Cloud Install](https://github.com/discourse/discourse/blob/master/docs/INSTALL-cloud.md)
- [Troubleshooting email on a new Discourse install](https://meta.discourse.org/t/troubleshooting-email-on-a-new-discourse-install/16326)
- [Configure Reply-By-Mail](https://meta.discourse.org/t/set-up-reply-via-email-support/14003)
- [Configure Backups](https://meta.discourse.org/t/configure-automatic-backups-for-discourse/14855)
- [Running Discourse alongside other websites](https://meta.discourse.org/t/running-other-websites-on-the-same-machine-as-discourse/17247)
- [Multiple Discourse Installs](https://linuxhandbook.com/multiple-discourse-install/)
- [Add offline page when rebuilding](https://meta.discourse.org/t/adding-an-offline-page-when-rebuilding/45238)
