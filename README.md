# Linux Server Configuration

This is the final project of Udacity Full Stack Nanodegree Course. It is particular to deploy Item-Catalog app on AWS Lightsail instance

* Public IP: 52.59.198.48

* SSH Port: 2200

* URL: http://52.59.198.48

* EC2 URL: "http://ec2-18-184-199-168.eu-central-1.compute.amazonaws.com"


## Paths

This project consists of four paths:

A- Configure ports and Firewall

B- Secure my app server and the machine

C- Prepare my app for deploying

D- Export my Lighsail instance to EC2 instance


## Path-A

1- Making sure the machine packages are up-to-date

  Run `sudo apt-get upgrade` `sudo  apt-get update`  

2- Changing SSH port from 22 to 2200 `sudo nano /etc/ssh/sshd_config`

3- Add SSH port 2200 to AWS to allow getting into machine after Changing ports

4- Disable all coming connections to machine and allow all out going on Firewall

 Run: `sudo ufw default deny incoming` `sudo ufw default allow outgoing`

5- Allow only needed ports to receive connections

  Run: `sudo ufw allow 2200/tcp ---> for SSH connections`

  Run: `sudo ufw allow 80/tcp ---> for HTTP connections`

  Run: `sudo ufw allow 123/udp --> for NTP connections`

  Run: `sudo ufw enable`

  Run: `sudo ufw status ---> To check allow ports`

## Path-B

1- Creating new user grader `sudo adduser grader --force-badname`

2- Add grader as sudoers

  Run:`sudo nano /etc/sudoers.d/grader`

  write inside: `grader ALL=(ALL:ALL)ALL`

3- Log in to new user `sudo su - grader`

4- Create key-pair encryption

  Run: `ssh-keygen` ---> On local machine

5- Making new dir .ssh and file .ssh/authorized_keys to save public keys on Linux and secure them by passphase

  Run: `mkdir .ssh`

  Run: `nano .ssh/authorized_keys`

  Run: `cat ~/key.pub file` ---> On local machine, Copy public keys and paste   them inside `.ssh/authorized_keys`

  Run: `chmod 700 .ssh chmod` then `chmod 644 .ssh/authorized_keys`

  Run: `sudo nano /etc/ssh/sshd_config` ---> Check that Password
       Authentication is (no) to force Key-based Authentication

6- Now to log in as grader `ssh -i ~/.ssh/secretkey -p 2200 grader@52.59.198.48`

7- Disable ubuntu user `sudo usermod --expiredate 1 ubuntu`

8- Disallowing root login `sudo nano /etc/ssh/sshd_config`

  change `PermitRootLogin prohibit-password` to `PermitRootLogin no`


## Path-C

1- Configure the local timezone to UTC `sudo dpkg-reconfigure tzdata`

2- Install Apache weberver `sudo apt-get install a2pache2`

3- Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`

4- Enable mod_wsgi `sudo a2enmod wsgi`

5- Intall git, make clone directory, then clone Item-Catalog app from github as sub-directory called catalog

  Run: `sudo apt-get install git`

  Run: `cd /var/www/`

  Run: `mkdir catalog`

  Run: `sudo git clone <URL> catalog`

6- Rename webserver.py to __init__.py

  Run: `sudo mv webserver.py __init__.py`

7- Setting up virtual environment to will keep the application and its        dependencies isolated from the main system

  Run: `cd /var/www/catalog/catalog`

  Run: `sudo apt-get install python-pip`

  Run: `sudo pip install virtualenv`

  Run: `sudo virtualenv venv` or `virtualenv --always-copy venv`

  Run: `source venv/bin/activate`

  Run: `sudo chmod -R 777 venv`

8- Test python files to check them running successfully and install any missing  modules

  Run: `sudo python __init__.py`

  Run: `sudo python database_setup.py`

  Run: `sudo python movieitems.py`  

  Run: `sudo python sqlalchemy oauth2client requests`

  Run: `pip install <package name>` ---> for any other missing modules

9- Create new virtual host for my application

  Run: `sudo nano /etc/apache2/sites-available/catalog.conf`

  Write inside:

       `<VirtualHost *:80>

      		ServerName 52.59.198.48

      		WSGIScriptAlias / /var/www/catalog/catalog.wsgi

      		<Directory /var/www/catalog/Catalog/>

      			Order allow,deny

      			Allow from all

      		</Directory>

      		Alias /static /var/www/catalog/catalog/static

      		<Directory /var/www/catalog/catalog/static/>

      			Order allow,deny

      			Allow from all

      		</Directory>

      		ErrorLog ${APACHE_LOG_DIR}/error.log

      		LogLevel warn

      		CustomLog ${APACHE_LOG_DIR}/access.log combined

       </VirtualHost>`

  Run: `sudo a2ensite catalog.conf` ---> To enable VH

10- Create .wsgi file ---> This file runs the server

  Run: `cd /var/www/catalog`

  Run: `sudo nano catalog.wsgi`

  Write inside:

     `import sys

      import logging

      logging.basicConfig(stream=sys.stderr)

      sys.path.insert(0,"/var/www/catalog/")

      from catalog import app as application

      application.secret_key = 'super_secret_key`

  Restart Apache

  Run:`sudo service apache2 restart` or `sudo service apache2 reload`

12- Install postgersql server

  Run: `sudo apt-get install postgresql`

  Run: `sudo su - postgres` ---> Log in as postgres adduser

  Run: `psql` ---> Get into DB

13- Create new database user with limited permissions to catalog application database

  Run: `CREATE USER catalog WITH PASSWORD "catalog";

             CREATE DATABASE catalog;`

  Check no remote connection to DB

  Run: `sudo cat /etc/postgresql/9.5/main/pg_hba.conf`

            `# Allow replication connections from localhost, by a user with the
             # replication privilege.
             #local   replication     postgres                                peer
             #host    replication     postgres        127.0.0.1/32            md5
             #host    replication     postgres        ::1/128                 md5`

14- Change DB in my app from SQLite to PSQL

  Run:

       `sudo nano /var/www/catalog/catalog/__init__.py

        sudo nano /var/www/catalog/catalog/database_setup.py

        sudo nano /var/www/catalog/catalog/movieitems.py`

  change `engine=create_engine('sqlite:///movies.db')`

   to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

15- Run App

  Run:

       `python /var/www/catalog/catalog/database_setup.py

        python /var/www/catalog/catalog/movieitems.py

        python /var/www/catalog/catalog/__init__.py`

16- Restart apache `sudo service apache2 reload`    

17- Vist site  IP: 52.59.198.48

18- Incase any errors appeared `sudo tail -f /var/log/apache2/error.log`


## Path-D

Lightsail instance doesn’t support domain name which is mandatory for Google oauth2 sign in so we go to EC2

1- Going to snapshots on lightsail instance and create one

* snapshot make a copy of all instance database

2- Export snapshot to Amazon EC2, launch it, and configure ports

3- Create new key pair

4- Adding URL to Authorized JavaScript origins on google credentials

5- Adding EC2 Domain to Google's authorized domain list

6- Update client_secrets file

  Run: `sudo nano /var/www/catalog/catalog/client_secrets`

  add EC2 URL to authorized JavaScript origins

  check `http://localhost:5000/gconnect` is in redirect_uris

7- Change ServerName in catalog.wsgi to EC2 URL

8- Change client_secrets path in __init__.py files

      `CLIENT_ID = json.loads(
       open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']

       oauth_flow = flow_from_clientsecrets('/var/www/catalog/catalog/client_secrets.json', scope='')`

## Error Handling

* When installing psycopg2 module:

       `ImportError: No module named psycopg2`

  Run: `pip install psycopg2-binary`


      `Error: You need to install postgresql-server-dev-X.Y for building a server-side extension or libpq-dev for building a client-side application.`

  Run:

            `sudo apt-get install postgresql

             sudo apt-get install python-psycopg2

             sudo apt-get install libpq-dev`

* When installing flask

       `ImportError: No module named flask`

  Run: `sudo nano /etc/apache2/sites-available/catalog.conf`

  Add the following two lines before the "WSGIScriptAlias" line:

       `WSGIDaemonProcess catalog python-path=/var/www/FlaskApp:/var/www/  catalog/catalog/venv/lib/python2.7/site-packages

        WSGIProcessGroup catalog`  

  Restart Apache with `service apache2 restart`


## Remove Rows from Items

1- log in to DB as postgres user `sudo su - postgres`

* `\l` is used to show and listing all DBs

2- Switch to catalog user `\c catalog`

3- In case deleting categories we will face `error: update or delete on table "category" violates foreign key constraint` error that is because category table is foreign key for movive table.

* This is called `Cascade Delete Error`

4- So we use command

       `DELETE FROM some_child_table WHERE some_fk_field IN (SELECT some_id FROM some_Table);  

        DELETE FROM some_table;`

* In this project command is:

       `DELETE FROM movie WHERE category_id IN (SELECT 12 FROM category);

        DELETE FROM category where id=12;`

## Third Party Resouces

1- [stackoverflow](https://askubuntu.com/questions/282806/how-to-enable-or-disable-a-user)

2- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

3- [stackoverflow](https://stackoverflow.com/a/42052504)

4- [stackoverflow](https://stackoverflow.com/a/28938258)
