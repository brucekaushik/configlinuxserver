LINUX SERVER CONFIGURATION PROJECT
=======================================

### Allow grader login:

login as the user with root priviledges and add user

sudo adduser grader

and follow the prompts to create the user

### Elevate 'grader' to have sudo permissions

open the following file

sudo nano /etc/sudoers.d/90-cloud-init-users

add the following lines to the end of file

# user rules for grader
grader ALL=(ALL) NOPASSWD:ALL

save and exit nano

restart sshd service by executing:

sudo service sshd restart

### Update packages on server

sudo apt-get update
sudo apt-get upgrade

### Generate ssh-public key for grader user (on your local computer)

execute the following steps:

ssh-keygen

give the path when prompted, leave the password as empty 

NOTE: I left the password empty for easy of use of reviewers, otherwise i could have set a password to protect the key file

in the location provided, copy the key from grader.pub for later use

### Authorize key on server

at /home/grader create '.ssh' directory by executing:

mkdir .ssh

inside .ssh directory create 'authorized_keys' file by executing:

touch authorized_keys

paste the key copied from grader.pub in the file & save it

set the right permissions on the server

sudo chmod 700 /home/grader/.ssh
sudo chmod 640 /home/grader/.ssh/authorized_keys

### Disable root login remotely

Open sshd config file:

sudo nano /etc/ssh/sshd_config

In the file, search for the following entry, and change 'yes' to 'no'

PermitRootLogin yes 

Restart ssdh service:

sudo service sshd restart

### Filewall configurations

By Default deny incoming and allow outgoing connections on all ports:

sudo ufw default deny incoming
sudo ufw default allow outgoing

Allow incoming connections for SSH, HTTP & NTP on ports specified:

sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp

Enable firewall:

sudo ufw enable

In case you need to check the status of the firewall:

sudo ufw status

### Enforce key based authentication:

Open sshd config file:

sudo nano /etc/ssh/sshd_config

In the file, search for the following entry, and change 'yes' to 'no'

PasswordAuthentication no 

Restart ssdh service:

sudo service sshd restart

### Change ssh port

Open sshd config file:

sudo nano /etc/ssh/sshd_config

In the file, search for the following entry, and change '22' to '2200'

Port 22 

Restart ssdh service:

sudo service sshd restart

### Install Dependencies

Install dependencies on server by executing the following commands:

sudo apt-get install apache2
sudo apt-get install python-setuptools
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install postgresql
sudo apt-get install python-psycopg2
sudo apt-get install python-pip
sudo pip install sqlalchemy
sudo pip install requests
sudo pip install flask
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install psycopg2 

### Setup Database

Create the database, database user, and grant the necessary priviledges by executing the following commands:

sudo su - postgres
psql
CREATE DATABASE itemcatalog;
CREATE USER itemcatalog;
ALTER ROLE itemcatalog WITH PASSWORD 'itemcatalog';
GRANT ALL PRIVILEGES ON DATABASE itemcatalog TO itemcatalog;
\q
exit

Additional Commands (for reference):

\l
\c
\dt

### Pull the code from the git repo (the one with wsgi version changes):

cd ~
git clone https://github.com/brucekaushik/itemcatalogwsgi.git ItemCatalog

Change path in wsgi file:

nano ~/ItemCatalog/vagrant/catalog/app.wsgi

in the line: sys.path.insert(0, "/home/grader/ItemCatalog/vagrant/")
change brucekaushik to grader

### Set Proper File Permissions:

sudo chmod -R 664 

sudo chown -R www-data:www-data /home/grader/ItemCatalog/vagrant/catalog
sudo find /home/grader/ItemCatalog/vagrant/catalog -type d -exec chmod 755 {} \;
sudo find /home/grader/ItemCatalog/vagrant/catalog -type f -exec chmod 644 {} \;

### Serve catalog application on port 80 

sudo > /etc/apache2/site-available/000-default.conf
sudo nano /etc/apache2/site-available/000-default.conf


Add the following lines and save the file:

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName 139.59.0.181
        ServerAlias 139.59.0.181
        DocumentRoot /home/grader/ItemCatalog/vagrant/catalog/
        ErrorLog /home/grader/ItemCatalog/vagrant/catalog/logs/error.log
        CustomLog /home/grader/ItemCatalog/vagrant/catalog/logs/access.log combined

        WSGIDaemonProcess ic user=www-data group=www-data threads=5
        WSGIProcessGroup ic
        WSGIScriptAlias / /home/grader/ItemCatalog/vagrant/catalog/app.wsgi
        Alias /static/ /home/grader/ItemCatalog/vagrant/catalog/static/

        <Directory /home/grader/ItemCatalog/vagrant/catalog>
                Require all granted
        </Directory>
</VirtualHost>
