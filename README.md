# From a Ubuntu 20.04 to Awesome Website in ~5 mins 
## A list of copy/paste-able commands (with short video walkthrough) to go from an Ubuntu 20.04 install on any IaaS provider to a basic yet beautiful website, all in about 5 mins.

How to spin up a basic yet beautiful, responsive, easily extensible website (on AWS, Azure, GCP, Digital Ocean, etc.) on a Custom Hostname, delivered via HTTPS and with SFTP access - all almost FREE and all within ~5 mins!

## WHY
I had a need to be able to spin up a basic lowest common demoninator version of a hello world type website on a variety of PaaS providers including AWS, Azure, GCP and Digital Ocean to test and demo Akamai technologies for my customers.  Once I understood the process I thought it would be useful to share in case someone had the same need I did in a way that was as quick, easy, and elegant as I could.

## HOW
This isn't a LAMP or LEMP or MERN stack, it's just NGINX, HTTPS, SFTP, and a basic yet gorgeous basic website of your choosing which only includes html, css, images.  From there it's up to you to extend it however you'd like by adding say PHP and mySQL and Wordpress if you'd like or whatever. 

## PREREQUISITES

1. AWS account (You'll have to put down a credit card, but free tier works)
2. Custom hostname (I use Google Domains for this at $12/yr for the hostname)
3. DNS resolver (I use Google Domains for this tool as part of what I already pay for but could be easily done with Edge DNS as well)
4. Command line tool (I use iTerm 2 but Terminal or Putty work also)

## INSTRUCTIONS

### 1. Setup Ubuntu 20.04 on an AWS EC2 micro instance

A. From within the AWS Console navigate to EC2 launch the instance

B. Name the key "key" and download it as a .pem, set it's file permissions to 600 by running `chmod 644 key.pem` then SSH in using key.pem for authentication.

C. Run...
```
sudo su
apt update
```

### 2. Setup Nginx on Ubuntu 20.04

A. Run...
```
apt install nginx
```
... then run...
```
ufw allow 80
ufw allow 443
ufw allow 22
curl -4 icanhazip.com
```
... then copy that IP and drop it into a browser.  You should see the Nginx Welcome page via HTTP (not HTTPS)

### 3. Setup a Custom Domain

A. Update the DNS of the custom hostname to include an A record pointing at the server's IP address.

B. Once this propagates, the same Nginx Welcome Page should be visible via the custom hostname via HTTP.

### 4. Setup a Cert for HTTPS delivery sing Let's Encrypt and Certbot

A. Run...
```
apt install letsencrypt
```
... then run...
```
systemctl status certbot.timer
```
... then run...
```
apt install python3-certbot-nginx
```
... then run...
```
certbot --nginx --agree-tos --preferred-challenges http -d YOUR-DOMAIN
```
... replacing YOUR-DOMAIN with your custom hostnamme.

B. From a browser navigate to the custom hostname and you should see the Nginx Welcome page via HTTPS instead of HTTP.

### 5. Setup SFTP 

A. Run...
```
apt-get install ssh-server -y
vi /etc/ssh/sshd_config
```

B. Comment out this...
```
Subsystem       sftp    /usr/lib/openssh/sftp-server
```
... like this...
```
#Subsystem       sftp    /usr/lib/openssh/sftp-server
```
... and change this...
```
PasswordAuthentication no
```
... to this...
```
PasswordAuthentication yes
```
... then add this at the end of the file...

```
Subsystem sftp internal-sftp
Match group ftpaccess
ChrootDirectory %h
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```
... then save.

C. Run...
```
addgroup sftp
useradd -m sftpUser -g sftp
passwd sftpUser
```
... then enter a new password and then run...
```
chmod 700 /home/sftpUser # /home/sftpUser
systemctl restart ssh
```
... then run...
```
sftp sftpUser@YOUR-IP
```
... replacing YOUR-IP with the server's IP address.  Next you should be prompted for the password you created and once you enter it, the prompt should change to sftp>.  From here you can just type `exit` to get out of sftp.

D. Run...
```
chmod 777 -R /var/www/html  
```
  
### 6. Access the server via SFTP using Filezilla
  
A. Download a beautiful static site template for free from https://html5up.net

B. SFTP in via Filezilla and upload the template
  
### 7. Navigate to your hostname via HTTPS and see the completed site ready to use as an origin for an Akamai CDN config.
