# Mailserver Setup

ip: 50.116.8.231

Linode VPS, 1GB Ram, 1CPU, Ubuntu 20.04.1 LTS

## Create DNS MX Record

    DNX Record Type     Name    Value           TTL
    MX                  @       mail.shiny.ooo  90sec

## Preparing Mail Server Installation

Creating mailadmin user

    adduser mailadmin
    adduser mailadmin sudo
    su - mailadmin

Do OS upgrades

    sudo apt update
    sudo apt upgrade -y

Set a fully qualified domain name (FQDN) for the server

    sudo hostnamectl set-hostname mail.shiny.ooo

Update /etc/hosts

    sudo nano /etc/hosts

Add the following

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

    sudo certbot certonly --webroot --agree-tos --email admin@shiny.ooo -d mail.shiny.ooo -w /var/www/html/

When it asks you if you want to receive communications from EFF, you can choose `No`.

The certificate and chain should be saved at /etc/letsencrypt/live/mail.shiny.ooo/

Configure Nginx

    sudo nano /etc/nginx/templates/ssl.tmpl

Find the following 2 lines.

    ssl_certificate /etc/ssl/certs/iRedMail.crt;
    ssl_certificate_key /etc/ssl/private/iRedMail.key;

Replace them with:

    ssl_certificate /etc/letsencrypt/live/mail.shiny.ooo/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mail.shiny.ooo/privkey.pem;

Save and close the file. Test nginx configuration and reload.

    sudo nginx -t
    sudo systemctl reload nginx


## Installing TLS Certificate in Postfix and Dovecot

Set up Postfix SMTP server and Dovecot IMAP server to use the Let’s Encrypt issued certificate.

Edit the main configuration file of Postfix.

    sudo nano /etc/postfix/main.cf

Find the following 3 lines. (line 95, 96, 97).

    smtpd_tls_key_file = /etc/ssl/private/iRedMail.key
    smtpd_tls_cert_file = /etc/ssl/certs/iRedMail.crt
    smtpd_tls_CAfile = /etc/ssl/certs/iRedMail.crt

Replace them with:

    smtpd_tls_key_file = /etc/letsencrypt/live/mail.shiny.ooo/privkey.pem
    smtpd_tls_cert_file = /etc/letsencrypt/live/mail.shiny.ooo/cert.pem
    smtpd_tls_CAfile = /etc/letsencrypt/live/mail.shiny.ooo/chain.pem

Save and close the file. Reload Postfix.

    sudo systemctl reload postfix

Edit the main configuration file of Dovecot.

    sudo nano /etc/dovecot/dovecot.conf

Find the following 2 lines. (line 47, 48)

    ssl_cert = </etc/ssl/certs/iRedMail.crt
    ssl_key = </etc/ssl/private/iRedMail.key

Replace them with:

    ssl_cert = </etc/letsencrypt/live/mail.shiny.ooo/fullchain.pem
    ssl_key = </etc/letsencrypt/live/mail.shiny.ooo/privkey.pem

Save and close the file. Reload dovecot.

    sudo systemctl reload dovecot

## Send Test Email

Log into the iredadmin panel with the postmaster mail account postmaster@shiny.ooo. In the Add tab, you can add additional domains or email addresses.

After you create an address, you can visit the Roundcube webmail address and login with the new mail user account.

    https://mail.your-domain.com/mail/

Test sending and receiving. For troubelshooting, [Errors you may see while maintaining iRedMail server](https://docs.iredmail.org/errors.html#recipient-address-rejected-sender-is-not-same-as-smtp-authenticate-username)

## Fail2ban Allowlist

Add the IP address of any connecting servers to the allowlist by editing the jail.local file.

sudo nano /etc/fail2ban/jail.local

Add the server IP address to the ignore list

    ignoreip = 127.0.0.1 127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 198.74.51.225/24

Save and close the file. Then restart Fail2ban.

    sudo systemctl restart fail2ban

## Improving Email Deliverablity

To prevent your emails from being flagged as spam, set PTR, SPF, DKIM and DMARC records.

### PTR Record

To check the PTR record for an IP address, run:

    dig -x IP-address +short

or

    host IP-address

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
    TXT                 dkim._domainkey     v=DKIM1l; p= MIIsd...   90sec

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

To create a DMARC record, go to your DNS manager and add a TXT record. In the name field, enter `_dmarc`. In the value field, enter the following. (Make sure to reate the dmarc@shiny.ooo email address)

    v=DMARC1; p=none; pct=100; rua=mailto:dmarc@shiny.ooo

    DNX Record Type     Name       Value                                                    TTL
    TXT                 _dmarc     v=DMARC1; p=none; pct=100; rua=mailto:dmarc@shiny.ooo    5min


### Auto Renew TLS Cert

Let’s Encrypt issued TLS certificate is valid for 90 days. We use Cron to automatically renew the certificate.

    sudo certbot renew -w /var/www/html/

--dry-run option to test the renewal process, instead of doing a real renewal.

    sudo certbot renew -w /var/www/html/ --dry-run

Edit the SSL virtual host /etc/nginx/sites-enabled/00-default-ssl.conf. Add the following lines.

    location ~ /.well-known/acme-challenge {
        root /var/www/html/;
        allow all;
    }

Save and close the file. Test Nginx configuration and reload.

    sudo nginx -t
    sudo systemctl reload nginx

### Create Cron Job

If the dry run is successful, you can create Cron job to automatically renew the cert. Open root user’s crontab file:

    sudo crontab -e

Then add the following line at the bottom of the file.

    @daily certbot renew -w /var/www/html/ --quiet && systemctl reload postfix dovecot nginx

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
