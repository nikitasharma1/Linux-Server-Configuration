# Linux Server Configuration Project

This project aims at configuring a linux server remotely to host a web application.

## Server Info

IP Address: ~~18.218.227.204~~

SSH Port: ~~2200~~

Application URL: ~~http://ec2-18-218-227-204.us-east-2.compute.amazonaws.com/~~

## Server Configuration

### Upgrade Packages 

Upgrade previously installed packages and autoremove the dependencies which are no longer required.

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo apt-get autoremove
```

### Install Utility tools

*finger* - user information lookup program
*tree* - displays directory tree

```
$ sudo apt-get install finger tree
```

### Configure local time zone to UTC

```
$ sudo dpkg-reconfigure tzdata 
```

From the pop up, select 'None of the above', then 'UTC'.

### Create user grader, give permission to sudo, create ssh key pair

1. Create user **grader**. Give the permission to **sudo**.
Create a file *grader* inside the directory *sudoers.d* with content as:
```grader ALL=(ALL:ALL) ALL```

```
$ sudo adduser grader
$ sudo nano /etc/sudoers.d/grader
```

2. Create an SSH key pair for grader using the ```ssh-keygen``` tool.
3. Place the public key on the server.
    - Inside grader's home directory, make directory *.ssh*.
    - Inside *.ssh*, make file *authorized_keys*.
    - Store the public key in this file.
    - Change the owner and permissions.

```
$ sudo mkdir /home/grader/.ssh
$ sudo nano /home/grader/.ssh/authorized_keys
$ sudo chmod 700 /home/grader/.ssh
$ sudo chmod 644 /home/grader/.ssh/authorized_keys
$ sudo chown -R grader:grader /home/grader/.ssh
```

4. Enforce key based login in */etc/ssh/sshd_config* with ```PasswordAuthentication no``` and disable root login with ```PermitRootLogin no```.

```
$ sudo service ssh restart
```

### Change ssh port to 2200 and configure Firewall

1. Update */etc/ssh/sshd_config* with ```Port 2200```.

```
$ sudo service ssh restart
```

2. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 80/tcp
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 123/udp
$ sudo ufw allow ntp
$ sudo ufw enable
```

### Install and configure Apache

Install apache and the corresponding application handler mod_wsgi for the python version (python 3 in this case). Disable the default site and reload the server to commit changes.

```
$ sudo apt-get install apache2 libapache2-mod-wsgi-py-3
$ sudo a2dissite 000-default
$ sudo service apache2 reload
```

### Install and configure Postgresql

1. Install postgresql and create user **catalog** and an empty database **catalog** under this user.

```
$ sudo apt-get install postgresql postgresql-contrib
$ sudo -u postgres createuser -P catalog
$ sudo -u postgres createdb -O catalog catalog
```

2. Check that the configuration file */etc/postgresql/9.5/main/pg_hba.conf* only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6.

### Install and configure git

```
$ sudo apt-get install git
$ git config --global user.name <username>
$ git config --global user.email <email>
```

### Install dependencies

```
$ sudo apt-get install python3-pip python3-psycopg2 python3-flask python3-sqlalchemy python3-dev

$ pip3 install flask httplib2 request requests oauth2client sqlalchemy virtualenv
```

### Create and configure virtual environment

```
$ sudo -p python3 virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
$ pip3 install flask httplib2 request requests oauth2client sqlalchemy
$ deactivate
```

### Deploy Project

1. Create a directory **catalog** inside */var/www/*. Update permissions.

```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
```

2. Clone the project repository inside the directory *catalog* as **catalog**. Switch to branch **lsc** inside the project repo.

```
$ cd /var/www/catalog
$ git clone https://github.com/nikitasharma1/Item-Catalog.git catalog
$ cd catalog
$ git checkout lsc
```

3. Update google oauth client secrets and the corresponding file "/var/www/catalog/catalog/client_secrets.json"

4. Create **catalog.wsgi** file inside */var/www/catalog/* with content as:

```
#!/usr/bin/python3

import sys
import logging

activate_this = '/var/www/catalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/catalog')

from catalog.project import app as application
application.secret_key = 'somekey'
```

4. Configure and enable new virtual host

Create a virtual host configuration file **catalog.conf** to serve the catalog application.

```
$ sudo nano /etc/apache2/sites-available/catalog.conf
```

The content for *catalog.conf* would be:

```
<VirtualHost *:80>
    ServerName catalogapp.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi

    WSGIDaemonProcess catalog python-path=/var/www/catalog/

    WSGIProcessGroup catalog

    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>

    Alias /static /var/www/catalog/catalog/static/
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the site (virtual host) to serve catalog app. Reload the server to commit changes.

```
$ sudo a2ensite catalog
$ sudo service apache2 restart 
```

## Acknowlegments
- [Udacity](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004/)
- [Udacity Discussion Forum](https://discussions.udacity.com/t/linux-server-configuration-internal-server-error/483779)
- [Amazon Lightsail](https://cloudacademy.com/blog/how-to-set-up-your-first-amazon-lightsail/)
- [POSTGRESQL TUTORIAL](http://www.postgresqltutorial.com)
- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [Ask Ubuntu](https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server)
- [Stack Overflow](https://stackoverflow.com)
- [stevewooding/fullstack-nanodegree-linux-server-config](https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config),
[iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)
