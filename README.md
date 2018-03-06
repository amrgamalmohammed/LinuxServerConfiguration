# LinuxServerConfiguration

This project aims to deploy the item catalog project on a linux server and configure the server to some security concerns.

- IP address ```217.55.11.168```
- SSH port ```2200```
- URL http://217.55.11.168.xip.io/

### SSH Access

- Private key generated locally using ```ssh-keygen``` (provided in the submission notes only).
- SSH directory on server is ```~/.ssh```.
- Use ```ssh grader@217.55.11.168 -p 2200 -i {path_to_key_file}```.

## Configurations made

### Creating new user

- Add grader user ```sudo adduser grader```.
- Grand sudo permissions ```sudo nano /etc/sudoers.d/grader``` and add line ```grader ALL=(ALL:ALL) ALL```.

### Update the machine

- ```sudo apt-get update```
- ```sudo apt-get upgrade```

### Setting SSH 

- Generate key on local machine ```ssh-keygen```.
- Create directory ```sudo mkdir .ssh``` then create file ```sudo nano .ssh/authorized_keys``` and copy the key contents to it.
- Change permissions ```chmod 700 .ssh``` ```chmod 644 .ssh/authorized_keys```
- Edit ```sudo nano /etc/ssh/sshd_config``` change port from 22 to 2200 and change ```PasswordAuthentication``` to no.
- Finally restart ssh service ```sudo service ssh restart```.

### Setting firewall

- Allow SSH (port 2200) ```sudo ufw allow 2200/tcp```.
- HTTP (port 80) ```sudo ufw allow 80/tcp```.
- NTP (port 123) ```sudo ufw allow 123/udp```.
- Finally start firewall ```sudo ufw enable ```.

### Setting timezone

- Use ```sudo dpkg-reconfigure tzdata``` and follow instructions.

### Installing PostgreSQL

- Install Postgresql using ```sudo apt-get install postgresql```.
- Enter Postgresql console ```sodu su - postgres``` then ```psql```.
- Create a catalog database and user ```CREATE DATABASE catalog;``` then ```CREATE USER catalog WITH PASSWORD 'password';```.
- Grant permissions and exit ```GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;```.
- Make sure no remote access is allowed in file ```sudo nano /etc/postgresql/9.3/main/pg_hba.conf```.

### Configuring environment

- ```sudo apt-get install apache2```
- ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
- ```sudo apt-get install git```
- ```sudo apt-get install python```
- ```sudo apt-get install python-pip```
- ```sudo pip install flask sqlalchemy oauth2client requests```
- ```sudo apt-get -qqy install postgresql python-psycopg2```

### Project ItemCategories edits

- Clone ItemCategory project ```git clone https://github.com/amrgamalmohammed/ItemCatalog.git```.
- Edit ```database_setup.py```, ```database_init.py``` and ```application.py``` in line ```create_engine('sqlite:///onlineshop.db')``` to ```create_engine('postgresql://catalog:password@localhost/catalog')``` to support the new Postgresql database.
- Run ```databse_setup.py``` then ```databse_init.py``` to setup and populate the database.
- Rename ```application.py``` using ```sudo mv application.py __init__.py```.
- Rename project directory to ```FlaskApp```.

### Setting FlaskAPP

- Create file ```sudo nano /etc/apache2/sites-available/FlaskApp.conf``` and add the following to it:
```
<VirtualHost *:80>
	ServerName 217.55.11.168.xip.io
	ServerAdmin amrgamalmohammed@gmail.com
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Setting wsgi

- Create flaskapp wsgi file ```sudo nano /var/www/FlaskApp/flaskapp.wsgi```.
- Edit the file to connect to the FlaskApp directory and set logging information to ```apache2``` logs
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")
from FlaskApp import app as application
application.secret_key = 'secret_key'
```

### Third party authentication

- Edit google ```client_secrets``` to use new Javascript origins and URI as my new server URL.

### Finally

- Restart apache2 server ```sudo service apache2 restart``` and headd to http://217.55.11.168.xip.io/

### Helpful content

- Used some search on https://www.digitalocean.com/community
