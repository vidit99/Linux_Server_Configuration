Project for limux server configuration of Full Stack Nanodegree programme offered by Udacity.

PROJECT OVERVIEW-->

We will take a baseline installation of a Linux server and prepare it to host your web applications. 
You will secure your server from a number of attack vectors, install and configure a database server, 
and deploy one of your existing web applications onto it.
Deploying the catalog app from Item catalog project, using the Apache web server, 
Flask Python framework, PostgreSQL.

Server details-->
IP address: 13.126.171.227
Accessible SSH port: 2200
URL to Catalog App so project is runnig live at: 
http://ec2-13-126-171-227.ap-south-1.compute.amazonaws.com/


Packages required for the project:
Git , Finger , Mod_wsgi , Python-psycopg2 , Virtualenv , Flask , Apache2 , PostgreSQL , Pip, etc.


Steps to setup project on a Ubuntu server:

Instructions for SSH access to the instance-->
Download RSA Key, restrict key access, and ssh into instance and the RSA key is not given here
while it is being given in the notes to reviewer section.

Launch the Amazon Lightsail terminal by making a instance there.
For that you must be logged into your Amazon Web Services account.
For reference visit it(https://lightsail.aws.amazon.com/) 
and press Create new instance of Ubuntu.
You will get your respective public IP address and Private IP. Now download the default key-pair and copy to /.ssh folder by:

Now run: chmod 600 ~/.ssh/key.pem
Now run: `ssh -i ~/.ssh/key.pem ubuntu@13.126.171.227` to create the instance on your terminal

To Create a new user named grader run command: sudo adduser grader

To give the grader the permission to sudo by running: `sudo visudo`
Now inside the file add `grader ALL=(ALL:ALL) ALL` below the root user under "#User privilege specification" and save the file
To add grader user type in `grader   ALL=(ALL:ALL) ALL` by running: `sudo nano /etc/sudoers.d/grader`
To add root type in `root   ALL=(ALL:ALL) ALL` by running: `sudo nano /etc/sudoers.d/root`


To update all the installed  current packages run `sudo apt-get update`
To install all the updates:`sudo sudo apt-get upgrade`

Now you have to change the SSH port from 22 to 2200 by running: nano /etc/ssh/sshd_config`
and so add `port 2200` below `port 22` and also change `PermitRootLogin prohibit-password` --> `PermitRootLogin no` 
It is done to disallow root login. Now change `PasswordAuthentication` from `no` to `yes`.
Now restart ssh service by running :  'sudo service ssh reload'


Create SSH keys and copy to server manually 
For it generate key pair by : ssh-keygen. Now save your keygen file in your ssh directory by: `/home/ubuntu/.ssh/item-catalog`
Now change the SSH port number configuration in Amazon lightsail in networking tab to 2200.
Now login into your grader account by: `ssh -v grader@*13.126.171.227* -p 2200`
Now make .ssh directory by :`mkdir .ssh`
Create a file to store key by: `touch .ssh/authorized_keys`
Now read the contents of public key by: `cat .ssh/item-catalog.pub`
Now copy the key and paste in the file you just created in grader `nano .ssh/authorized_keys`)
Now set permissions by: `chmod 700 .ssh`
and  `chmod 644 .ssh/authorized_keys`
Now again change `PasswordAuthentication` from `yes` back to `no`. by applying  `nano /etc/ssh/sshd_config`
Now login with the key pair: `ssh grader@13.126.171.227* -p 2200 -i ~/.ssh/item-catalog`


Now configure the Uncomplicated Firewall (UFW) to only allow  incoming connections 
for SSH (port 2200), HTTP (port 80),  and NTP (port 123)
`sudo ufw status`
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow ssh`
`sudo ufw allow 2200/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/udp`
`sudo ufw enable`

Configure the local timezone to UTC by run: `sudo dpkg-reconfigure tzdata` 
Now select Asia and then Kolkata.


Install and configure Apache to serve a Python mod_wsgi application
`sudo apt-get install apache2` 
`sudo apt-get install libapache2-mod-wsgi`
Now configure the Apache to handle requests using WSGI module by: `sudo nano /etc/apache2/sites-enabled/000-default.conf`
Now add `WSGIScriptAlias / /var/www/html/myapp.wsgi` before `</VirtualHost>` that is the last line
Now restart the Apache server by running: `sudo apache2ctl restart`


Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
Now install git by :`sudo apt-get install git`

Install python dev and verify WSGI is enabled
`sudo apt-get install python-dev`
To verify wsgi is enabled run: `sudo a2enmod wsgi`
`cd /var/www` now run `sudo mkdir catalog` now run `cd catalog`
`sudo mkdir catalog` now run `cd catalog` now run `sudo mkdir static templates` now run `sudo nano __init__.py `

```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hey vidit"
if __name__ == "__main__":
    app.run()
```

Install flask
`sudo apt-get install python-pip`
`sudo pip install virtualenv `
`sudo virtualenv venv`
`sudo chmod -R 777 venv`
`source venv/bin/activate`
`pip install Flask`
`python __init__.py`
`deactivate`

To configure And Enable New Virtual Host : create config file by:
`sudo nano /etc/apache2/sites-available/catalog.conf`
Now paste the below code 

```
<VirtualHost *:80>
  ServerName 13.126.171.227
  ServerAdmin admin@13.126.171.227
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

`sudo a2ensite catalog`

Create the wsgi file by: `cd /var/www/catalog` now run: `sudo nano catalog.wsgi`

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key here'
```


Now run: `sudo service apache2 restart`

Now clone your Github Repo `sudo git clone https://github.com/vidit99/item_catalog_project`
Now make sure you get hidden files iin move `shopt -s dotglob`. 
Move files from clone directory to catalog `mv /var/www/catalog/item_catalog_project/* /var/www/catalog/catalog/`
Now remove the clone directory by: `sudo rm -r devpost`

Now make .git inaccessible
Do it by running `cd /var/www/catalog/` 
Now create .htaccess file `sudo nano .htaccess`
`RedirectMatch 404 /\.git`


Now install dependencies:
`source venv/bin/activate`
`pip install httplib2`
`pip install requests`
`sudo pip install --upgrade oauth2client`
`sudo pip install sqlalchemy`
`pip install Flask-SQLAlchemy`
`sudo pip install python-psycopg2`


Install and configure PostgreSQL:
`sudo apt-get install postgresql`
`sudo apt-get install postgresql-contrib`
`sudo nano database_setup.py`
replace below line in database_setup.py 
`python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')`
Now repeat the above for project.py
Now copy your project.py file into the __init__.py fileby:
`mv project.py __init__.py`
Add catalog user `sudo adduser catalog`
Now login as postgres super user by: `sudo su - postgres`
Now enter postgres by: `psql`
Now create user catalog by: `CREATE USER catalog WITH PASSWORD 'db-password';`
Now change the role of catalog to creatDB` ALTER USER catalog CREATEDB;`
run `\du` to display users
Now create new DB "catalog" with own of catalog by:
`CREATE DATABASE catalog WITH OWNER catalog;`
Now connect to database catalog by: `\c catalog`
Now revoke all the rights by: `REVOKE ALL ON SCHEMA public FROM public;`
Now give access to only catalog role by: `GRANT ALL ON SCHEMA public TO catalog;`
Now quit postgres by: `\q`
Now logout from postgres by : `exit`
Now setup your database schema by :`python database_setup.py`


Now google wont allow the IP address to make redirects so there is a need to set up the host name address to be used.
go to http://whatismyipaddress.com/ip-hostname and get host name by entering ip address
run: `sudo nano /etc/apache2/sites-available/catalog.conf`
Noe add `ServerAlias YOURHOSTNAME` below the `ServerAdmin`
Now run :`sudo a2ensite catalog`
Now restart the apache server by: `sudo service apache2 restart`
Now go tour google developer console
add your hostname->[ http://ec2-13-126-171-227.ap-south-1.compute.amazonaws.com/ ] 
and 13.126.171.227->IP address to Authorized Javascript origins. 
and add hostname/oauth2callback -> http://ec2-13-126-171-227.ap-south-1.compute.amazonaws.com/ouath2callback to Authorized redirect URIs.
and download json and copy it to the clonned json file.