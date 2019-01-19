# Linux Project

A baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## How to open the server

* URL of Project: [Catalog Item ](http://ec2-3-121-109-184.eu-central-1.compute.amazonaws.com/).
* Public IP Address: 3.121.109.184
* SSH port: 2200

## Amazon Lightsail Set Up
1. Click get started for free
2. Create your first instance
3. select linux/unix platform and OS only blueprint.






## Steps to Configure Linux server
1. downloaded .pem key from lightsail into folder .ssh 
2.  make key secure by  type ```chmod 600 ~/.ssh/[yourkey].pem```
3. log into the server as the user ```ubuntu``` ssh -i ~/.ssh/```[yourkey]```.pem ubuntu@```[your public ip]```
4. create a user named grader by ``` sudo adduser grader```
5. give the user grader superuser privileges by type ```sudo nano /etc/sudoers.d/grader```.
6. updating the package list and upgrade it.
7. install finger.
8. create an SSH Key by ```ssh-keygen``` by ```ssh-keygen -f ~/.ssh/[yourkey.rsa```.
9. in another terminal type ```cat ~/.ssh/[yourkey].rsa.pub``` to read your public key and copy this key.
10. Run the command cd /home/grader to move to the folder ```grader```.
11. Create a directory called .ssh with the command ``` mkdir .ssh```.
12. Edit that file using ```nano .ssh/authorized_keys``` and paste the key that copy it on step 9.
13. we need to configure the firewall by type:
``` 
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```

## Application Deployment

1. installing the required software
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo apt-get install git
```
2. Enable mod_wsgi by type ```sudo a2enmod wsgi``` then restart Apache ```sudo service apache2 restart```.
3. make the grader is owner by type 
```
cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog
```
4.cloning our Catalog Application repository by ```git clone [repository url] catalog```.
5. Create the .wsgi file by ``` sudo nano catalog.wsgi``` then paste this 
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
6. Rename your  project.py, or whatever you called it in your catalog application folder to __init__.py by ``` mv project.py __init__.py ```
7. create our virtual environment
```
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
```
8. install all packages required
```
sudo apt-get install python-pip
sudo pip install flask
sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...
```
9. client_secrets must be changed to its complete path : /var/www/catalog/catalog/client_secrets.json in ``` __init__.py ```

10. Run the database_setup.py and dummyBooks.py once to setup database with dummy data:
``` 
python database_setup.py
python films.py
 ```

10. ``` sudo nano /etc/apache2/sites-available/catalog.conf ```
and paste the following 
```
<VirtualHost *:80>
    ServerName [Public IP]
    ServerAlias [Hostname]
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
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
</VirtualHost>
```
11. set database
```
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres -i
psql
```
12. Create a database user and password

```
postgres=# CREATE USER catalog WITH PASSWORD [password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
exit
```
13. Now use nano again to edit your files that use engine

change the database engine from ```sqlite://catalog.db``` to ```postgresql://username:password@localhost/catalog```
14. sudo service apache2 restart
15. sudo service apache2 reload

## References:
* [https://github.com/mulligan121/Udacity-Linux-Configuration](https://github.com/mulligan121/Udacity-Linux-Configuration)
