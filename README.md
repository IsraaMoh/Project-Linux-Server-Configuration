# Project-Linux-Server-Configuration
This project is 5th project of Udacity's Full Stack Web Developer Nanodgree

this README walks through how to prepare a Linux server to host a web application.

 ip : 52.29.32.59
 port :2200
 url : http://52.29.32.59.xip.io/
private ip : 172.26.10.66
Ubuntu 16.04 LTS, Linux
Amazon Lightsail. server
PostgreSQL database server.
item catalog project.

# Get server
start new ubuntu linux server instance on Amazon Lightsail
# SSH into the server
from account - ssh keys download the default private key.
move the downloaded file into the local folder ~/.ssh and rename it Israa_key.rsa
in your terminal change the file permission by this command:chmod 600 ~/.ssh/israa_key.rsa
to connect to the instance via the terminal type: ssh -i ~/.ssh/israa_key ubuntu@52.29.32.59, 52.29.32.59 is the instance public IP.
# Secure the server 
- Update all currently installed packages.
sudo apt-get update
Upgrade all currently installed packages
sudo apt-get upgrade

- Change the SSH port from 22 to 2200
edit sshd_config file using sudo nano /etc/ssh/sshd_config
change the port number from 22 to 2200.
save the file using ctrl x then click y
restart SSH:
sudo service ssh restart.

# Configure the Uncomplicated Firewall
Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

check status:
sudo ufw status >> it should be inactive.
deny all incoming traffic:
sudo ufw default deny incoming
Enable outgoing traffic:
sudo ufw default allow outgoing
Allow incoming tcp packets on port 2200:
sudo ufw allow 2200/tcp
Allow incoming udp packets on port 123:
sudo ufw allow 123/udp
Allow HTTP traffic in :
sudo ufw allow www
Deny tcp and udp packets on port 22:
sudo ufw deny 22
Turn ufw on via:
sudo ufw enable
in the amazon server :
-to change the instance settings too click on the manage option of the Lightsail instance.
-click on Networking tab and change the fire wall settings like before in the server, allow SSH (port 2200), HTTP (port 80)

# Give grader access.
- Create a new user account named grader
while logged in as ubuntu : 
sudo adduser grader.
enter password and fill the information you can skip them by pressing enter, the only important thing is the password.

- Give grader the permission to sudo.
edit the sudoers file with sudo visudo.
look for the line that have root ALL=(ALL:ALL) ALL
copy and paste it with the name of the new user grader the line should look like this
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
save the file and exit

-verify grader sudo permissions. 
login as grader: su - grader 
type password
then type sudo -l
type password 

# Create an SSH key pair for grader using the ssh-keygen tool.
On the local machine:
Run ssh-keygen.
enter the file name to save the key i saved mine in the home directory.
run cat ~/.ssh/israa_key.pub then copy its contents.
login as grader in virtual machine.
run
  mkdir .ssh 
  sudo nano ~/.ssh/authorized_keys
 chmod 700 .ssh
 chmod 644 .ssh/authorized_keys
 
set /etc/ssh/sshd_config file for :
PasswordAuthentication and set it to no.
PermitRootLogin no
ProhibitRootLogin no
check everything is up to date
sudo apt-get update 
sudo apt-get upgrade
sudo service ssh restart

# Prepare to deploy your project.
- Configure the local timezone to UTC.
While logged in as grader, configure the time zone to UTC: sudo dpkg-reconfigure tzdata.
Current default time zone: 'Etc/UTC'
Local time is now:      Thu Dec 27 00:20:03 UTC 2018.
Universal Time is now:  Thu Dec 27 00:20:03 UTC 2018.

# Install and configure Apache to serve a Python mod_wsgi application.
while logged in as grader.
- install Apache:
sudo apt-get install apache2.
Enter public IP of the Amazon Lightsail instance into browser.
If Apache is working, you should find the Apache Ubuntu default page.
- my project is built with python3 so python3 mod_wsgi pakage need to be installed : 
sudo apt-get install libapache2-mod-wsgi-py3.
Enable mod_wsgi via: sudo a2enmod wsgi.
- Install and configure PostgreSQL:
install PostgreSQL:
sudo apt-get install postgresql

Restart SSH:
sudo service ssh restart

Switch to the postgres user:
sudo su - postgres.

Open PostgreSQL interactive terminal with > psql.

Create the catalog user with a password and give them the ability to create databases:

CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
ALTER ROLE catalog CREATEDB;

Exit psql: \q

Give to catalog user the permission to sudo:
sudo visudo

Below this line, add a new line to give sudo permissions to catalog user. the file should look like this:

root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL

save and exit

to verify catalog sudo permissions. login as grader: su - catalog , type password, then type sudo -l.

while logged in as catalog.
create database via : createdb catalog.
Exit psql: \q

# Installing and confguration
# Install Git: 
 sudo apt-get install git

Set your name, e.g. for the commits:
git config --global user.name "YOUR NAME"

Set up your email address to connect your commits to your account: 
git config --global user.email "YOUR EMAIL ADDRESS"
create /var/www/catalog/ directory.
clone this repo ''' git clone https://github.com/IsraaMoh/Catalog.git'''

cd to that directory and clone item catalog repository.

sudo git clone https://github.com/HatoonMo/ItemCatalogProject.git

From the /var/www directory, change the ownership of the catalog directory to grader via: sudo chown -R grader:grader catalog/

Cd to the /var/www/catalog/Catalog directory.

Rename the app.py file to __init__.py using: mv application.py __init__.py.
In __init__.py, replace this line : app.run(host='0.0.0.0', port=5000) to  app.run()

in each file replace this line  engine = create_engine('sqlite:///catalogDB.db') to engine create_engine('postgresql://catalog:catalog@localhost/catalog')

# Authenticating login through google
From Google Cloud Plateform.
Click APIs & services on left menu then Credentials.
Create an OAuth Client ID, add and add http://52.29.32.59.xip.io as authorized JavaScript origins.
in authorized redirect URI add 
http://52.29.32.59.xip.io/oauth2callback
http://52.29.32.59.xip.io/login
http://52.29.32.59.xip.io/gconnect .
Download the corresponding JSON file, open it then copy the contents and paste them into /var/www/catalog/Catalog/client_secrets.json.
Replace the client ID  of templates/login.html file in the project directory.
Add the path of client_secrets.json to  __init__.py ""/var/www/catalog/Catalog/client_secrets.json""

# Install virtual environment and all the project dependencies

install pip
Install the virtual environment: sudo apt-get install python-virtualenv

Change to the /var/www/catalog/Catalog/ directory.

Create the virtual environment:
sudo virtualenv -p python3 venv3.

Change the ownership to grader with:
sudo chown -R grader:grader venv3/.

Activate the new environment: 
. venv3/bin/activate.

- Install the following dependencies:

pip install flask
pip install sqlalchemy
pip install requests
pip install psycopg2
pip install --upgrade oauth2client

# set up virtual host
IN  /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:
<VirtualHost *:80>
   ServerName 52.29.32.59
   ServerAlias 52.29.32.59.xip.io
   WSGIScriptAlias / /var/www/catalog/catalog.wsgi
   <Directory /var/www/catalog/Catalog/>
       Order allow,deny
         Allow from all
   </Directory>
   Alias /static /var/www/catalog/Catalog/static
   <Directory /var/www/catalog/Catalog/static/>
         Order allow,deny
         Allow from all
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

Add the following line in /etc/apache2/mods-enabled/wsgi.conf file to use Python 3.
       WSGIPythonPath /var/www/catalog/Catalog/venv3/lib/python3.5/site-packages
       
enable virtual host:
sudo a2ensite catalog
sudo service apache2 reload

 # set up the database and fill it with data
cd to /var/www/catalog/Catalog/ directory.
activate the virtual environment via :
. venv3/bin/activate.
Run database_setup.py first
Run database_init.py second
deactivate virtual environment.

# start the web application
to disable the default Apache site.
sudo a2dissite 000-default.conf
sudo service apache2 reload
sudo service apache2 restart.
open in browser http://52.29.32.59.xip.io
       
       
