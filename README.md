project 5
=============

# Linux Server Configuration
This is a project for Udacity full stack web developer

IP Address : 54.93.123.163
Host Name  : ec2-54-93-123-163.eu-central-1.compute.amazonaws.com

password grader user login : udacity
password grader sudo : grader0000

# Prerequisites
- Amazon Lightsail Server Set Up :
1- Use this link to help you open amazon lightsail account : https://lightsail.aws.amazon.com/  
2-Create a new AWS account and then log in ageain from link above and press Create Instance .
3-Choose an instance image: Ubuntu
4-Give your instance a hostname and then Wait for it to start up.
5-Click the status card , and then click the Account page at the bottom.
6-Click the 'Download' to download your private key from Account page , we use it to log into our server . 
7-Click the 'Networking' and find the 'Add another' at the bottom. to add port 123 and 2200 .
8-move this downloaded .pem key into .ssh folder.

- Linux Configuration :
1-From here we will log into the server as the user ubuntu with our key. From the terminal type $ ssh -i ~/.ssh/udacity.pem ubuntu@54.93.123.163
2-create user " grader " ,from the command line $ sudo adduser grader .
3-$ sudo nano /etc/sudoers.d/grader , grader ALL=(ALL:ALL)ALL 
4- update package list 
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
5-install Finger tool $ sudo apt-get install finger , This tool will allow us to see the users on this server
7-Now we must create an SSH Key for our new user grader. From a new terminal run the command: $ ssh-keygen -f ~/.ssh/udacity.rsa
8-Create a directory called .ssh $ mkdir .ssh
9-Create a file to store the public key $ touch .ssh/authorized_keys , Edit this file using $ nano .ssh/authorized_keys , then paste in the public key .
10-restart its service with $ sudo service ssh restart ,  login to grader account using ssh. From terminal  $ ssh -i ~/.ssh/udacity.rsa grader@54.93.123.163
11-ssh configuration file by editing $ sudo nano /etc/ssh/sshd_config. Find the line that says PasswordAuthentication and change it to no.
 Also find the line that says Port 22 and change it to Port 2200. Lastly change PermitRootLogin to no
12- we need to configure the firewall using these commands:
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable
13- then we can login by , ssh -i ~/.ssh/udacity.rsa grader@54.93.123.163 -p 2200


- Application Deployment :
1-Start by installing the required software
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git

2-Enable mod_wsgi with the command $ sudo a2enmod wsgi , and restart Apache $ sudo service apache2 restart >
3-create a directory for our catalog application and make the user grader the owner.
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
4-clone Catalog Application repository github .
5-Create the .wsgi file by $ sudo nano catalog.wsgi
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'super_secret_key'

6-Rename my application.py to __init__.py by $ mv application.py __init__.py
7-create our virtual environment, make sure you are in /var/www/catalog.
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
8-enable our virtual host to run the site

$ sudo nano /etc/apache2/sites-available/catalog.conf
Paste in the following:

<VirtualHost *:80>
    ServerName 54.93.123.163
    ServerAlias ec2-54-93-123-163.eu-central-1.compute.amazonaws.com
    ServerAdmin admin@54.93.123.163
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

9-setting up the database

$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql

10- Create a database user and password
postgres=# CREATE USER catalog WITH PASSWORD 0000;
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit

11- Restart apache server $ sudo service apache2 restart

12- link to find host name  http://www.hcidata.info/host2ip.cgi





 



