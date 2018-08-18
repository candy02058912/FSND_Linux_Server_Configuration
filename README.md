# FSND Project - Linux Server Configuration

Catalog App Url: 

## Basic Information
* IP: 13.124.157.252
* SSH PORT: 2200
* Application URL: http://13.124.157.252.xip.io/

## Configuration Steps

### 1. Connect to server (On Mac)
* Download the private key from Lightsail
* Move file into `~/.ssh`
* Set permissions to 600:
```
chmod 600 <key-filename>
```
* Start ssh-agent by running:
```
eval "$(ssh-agent -s)"
```
* Add key to ssh-agent:
```
ssh-add -K <key-filename>
```
* Connet to server:
```
ssh ubuntu@13.124.157.252
```

### 2. Update packages
1. Update:
```
sudo apt-get update
```
2. Upgrade:
```
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### 3. Setup user `grader`
1. Install `finger` to see user info easily
```
sudo apt-get install finger
```
2. Create user `grader`
```
sudo adduser grader
```
3. Give sudo access to `grader`
Copy the ubuntu default file
```
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```
Change `ubuntu` to `grader`
```
sudo vim /etc/sudoers.d/grader
```
4. Create ssh keys for `grader` (On Mac)
```
ssh-keygen -t rsa -b 4096 -C "grader@13.124.157.252"
```
5. Add public key to `/home/grader/.ssh/authorized_keys`
```
sudo mkdir /home/grader/.ssh
sudo touch /home/grader/.ssh/authorized_keys
sudo vim /home/grader/.ssh/authorized_keys
```

### 4. Set ssh port from 22 to 2200
1. Change `PORT` from 22 to 2200
```
sudo vim /etc/ssh/sshd_config
```
2. Restart service
```
sudo service ssh restart
```
3. Change Firewall on Lightsail console

From:

![22](https://i.imgur.com/Bdhqev2.png)

To:

![2200](https://i.imgur.com/uXGQwMm.png)

### 5. Set locale (see Resources)
1. Insert into `/etc/default/locale`
```
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```
2. Generate missing locales
```
sudo dpkg-reconfigure locales
```

### 6. Configure firewall
1. Check firewall status
```
sudo ufw status
```
2. Block all incoming connections
```
sudo ufw default deny incoming
```
3. Allow all outgoing connections
```
sudo ufw default allow outgoing
```
4. Allow tcp connections on 2200 (ssh)
```
sudo ufw allow 2200/tcp
```
5. Allow tcp connections on 80 (HTTP)
```
sudo ufw allow 80/tcp
```
6. Allow udp connectsion on 123
```
sudo ufw allow 123/udp
```
7. Enable firewall
```
sudo ufw enable
```
8. Setup firewall on Lightsail console
![firewall](https://i.imgur.com/BtiLthS.png)


### 8. Setup postgresql and modify application code
1. Install poastgresql
```
sudo apt-get install postgresql
```
2. Switch to `postgres` account
```
sudo -i -u postgres
```
3. Access postgres prompt
```
psql
```
4. Create database and user
```
CREATE DATABASE catalog;
CREATE USER catalog;
```
5. Set password for user catalog
```
ALTER ROLE catalog WITH PASSWORD 'catalogpassword';
```
6. Setup permissions
```
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
7. Exit postgres prompt
```
\q
```
8. Clone the application and move into `/var/www/Catalog/Catalog`,
note that it is not called `Stuffed-animals-catalog`
```
git clone https://github.com/candy02058912/Stuffed-animals-catalog.git
```
8. Modify application code, replace all
```
create_engine('sqlite:///stuffedanimals.db')
```
with
```
engine = create_engine('postgresql://catalog:catalogpassword@localhost/catalog')
```

### 9. Setup server and application
1. Install apache server
```
sudo apt-get install apache2
```
2. Install mod_wsgi
```
sudo apt-get install libapache2-mod-wsgi
```
3. Enable mod_wsgi
```
sudo a2enmod wsgi 
```
4. Add the following to `/etc/apache2/sites-available/Catalog.conf`
```
<VirtualHost *:80>
    ServerName 13.124.157.252
    ServerAdmin candy02058912@gmail.com
    WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
    <Directory /var/www/Catalog/Catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/Catalog/Catalog/static
    <Directory /var/www/Catalog/Catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
5. Enable site
```
sudo a2ensite Catalog
```
6. Install unzip for unzipping `client_secrets.zip`
```
sudo apt-get install unzip
```
7. Install packages
```
sudo pip install Flask httplib2 sqlalchemy requests oauth2client psycopg2
```
8. Rename `application.py` to `__init__.py`
```
sudo mv application.py __init__.py
```
9. Setup initial database
```
sudo python database_setup.py
sudo python database_initialize.py
```
10. Add the following code to `/var/www/Catalog/catalog.wsgi`
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Catalog/")

from Catalog import app as application
application.secret_key = 'horrified-by-this'
```
11. Restart server
```
sudo apache2ctl restart
```

### Add `http://13.124.157.252.xip.io/` to Google Oauth authorized javascript origins

## Notes
* Password Authentication was `no` by default

## Resources
* https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://askubuntu.com/questions/881742/locale-cannot-set-lc-ctype-to-default-locale-no-such-file-or-directory-locale/895186
* http://docs.sqlalchemy.org/en/latest/core/engines.html
* https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04