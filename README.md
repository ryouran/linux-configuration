# Linux Server Configuration

Completed for [Udacity's Full Stack Web Developer Nanodegree program](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

## Project Overview

The goal of this project is to configure a Linux server, install and configure a database server, and deploy a web application onto it.  

I followed the instructions [here](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e) and created an Ubuntu Linux Server instance using [Amazon LightSail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) and deployed my Homework Tracker app onto it.

The app is available at [http://ec2-34-214-237-115.us-west-2.compute.amazonaws.com/](http://ec2-34-214-237-115.us-west-2.compute.amazonaws.com/)

### Server Information

Public IP address: 34.214.237.115

### Server Configuration

#### Update installed packages

- Download the package list from the repositories with the command.
	
	`sudo apt-get update`

- Install the latest versions of the packages with the command 

	`sudo apt-get upgrade`

#### Login to the Linux instance via SSH connection

- Download the default private key and save the .pem file to `~/.ssh` directory. 

- Change permission to make the private key more secure.

	`chmod 600 ~/.ssh/<yourPrivateKeyFile>.pem`

- Log in to the Linux instance from the terminal with the following command.

	`ssh -i ~/.ssh/<yourPrivateKeyFile>.pem ubuntu@34.214.237.115`


#### Add grader user and give the permissino to sudo

- Install to finger to check the user status

	`sudo apt-get install finger`

 Create a new user account named grader with the following command.

   `sudo adduser grader`

- Create a new file in the `/etc/sudoers.d` directory with the following command.

	`sudo nano /etc/sudoers.d/grader`

- Add the following into the grader file to provide the root privileges to grader and save it.  
	
	`grader ALL=(ALL:ALL) ALL`

- Create .ssh directory and authorized keys file for grader.

	`sudo mkdir /home/grader/.ssh`
	
	`sudo touch /home/grader/.ssh/authorized_keys`

- Change permission of .ssh directory and the authorized_keys file.

	`sudo chmod 700 /home/grader/.ssh`

	`sudo chmod 644 /home/grader/.ssh/authorized_keys`

- Change the owner from root to grader.

	`sudo chown -R grader:grader /home/grader/.ssh`

- Generate an SSH key pair on your local machine for grader with the following command.  The following command creates the key pair in the `~/.ssh/` directory.

	`ssh-keygen -f ~/.ssh/grader`

- Store the generated public key to the remote Linux instance manually.

	* Change directory to `~/.ssh/` on your local machine and copy the content of grader.pub file

	* Login to the remote server again if you have logged out as ubuntu user

		`ssh -i ~/.ssh/<yourPrivateKeyFile>.pem ubuntu@34.214.237.115`

	* Paste the content of <YourKeyName>.pub file into the authorized_keys file on the Linux instance.
		
		`sudo nano /home/grader/.ssh/authorized_keys`

	* Check if you can login as a grader.

		`ssh -i ~/.ssh/grader grader@534.214.237.115`


#### Change the SSH port from 22 to 2200. 

* Open sshd_config with the following command.

	`sudo nano /etc/ssh/sshd_config`

* Edit the Port from 22 to 2200 and save the file.

* Restart the SSH service

	`sudo service ssh restart`

* Configure the Uncomplicated Firewall (UFW) to allow incomming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) with the following commands.

	`sudo ufw allow 2200/tcp`

	`sudo ufw allow 80/tcp`

	`sudo ufw allow 123/udp`

	`sudo ufw enable`

* Log into the Linux instance via SSH with Port 2200.

	`ssh -i ~/.ssh/<YourPrivateKey>.pem -p 2200 ubuntu@34.214.237.115`

#### Diable password authentication and disable ssh login for root user

- Open sshd_config with the following command.

	`sudo nano /etc/ssh/sshd_config`

- Find the line which says "PasswordAuthentication" and ensure that it's set to 'no.'

- Restart SSH service with the following command.

	`sudo service ssh restart`

#### Configure the local timezone to UTC

- Run the following command

	`sudo dpkg-reconfigure tzdata`

	[https://help.ubuntu.com/community/UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)

#### Deploy the Item Catalog project


- Install and configure Apache.

	`sudo apt-get install apache2`

- Install Git (This has already been installed on the Linux instance).

	`sudo apt-get install git`

- Clone the Item Catalog project from the Github repository.

	* Clone the Item Catalog project from the repository.

		```
		sudo mkdir /var/www/catalog
		cd /var/www/catalog
		sudo git clone https://github.com/ryouran/item-catalog.git catalog
		```

	* Change the ownder of the folder

		`sudo chown -R grader:grader /var/www/catalog`

- Install mod_wsgi and create catalog.wsgi

		`sudo apt-get install libapache2-mod-wsgi`

		`sudo nano /var/www/catalog/catalog.wsgi`

	* Add the following into catalog.wsgi.

		```
		#!/usr/bin/python

		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, '/var/www/catalog/')

		from catalog import app as application
		application.secret_key = 'super_secret_key'
		```

		Source: [http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)

#### Create virtual environment for the catalog app.

- Install, create, and activate virtual environment in the `/var/www/catalog/catalog` directory

	```	
	sudo apt-get install python-pip
	sudo pip install virtualenv
	sudo virtualenv venv
	sudo chmod -R 777 venv
	source venv/bin/activate
	```

	＊Note: (venv) should appear in the prompt.


	Source: [http://flask.pocoo.org/docs/0.12/installation/](http://flask.pocoo.org/docs/0.12/installation/)

- Install packages in the virtual environment.

	```
	sudo pip install Flask
	sudo pip install httplib2 oauth2client sqlalchemy requests psycopg2-binary
	```

- Rename application.py to __init__.py in `/var/www/catalog/catalog` directory.

	`sudo mv application.py __init__.py`

- Replace this line app.run(0.0.0.0, port=5000) with app.run()

- Check if installation is successful.

	`sudo python __init__.py`


	Source: [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)



#### Configure and enable a new virtual host

- Change directory to the `/etc/apache2/sites-available/` folder.

- Copy 000-default.conf and create catalog.conf.

	`sudo cp 000-default.conf catalog.conf`

- Edit the content of catlaog.conf.

	`sudo nano /etc/apache2/sites-available/catalog.conf`

- Look up domain name 

	`nslookup 34.214.237.115`

- Edit the content as follows:


	```
	<VirtualHost *:80>
			ServerName 34.214.237.115
	        ServerAlias http://ec2-34-214-237-115.us-west-2.compute.amazonaws.com
	        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	        <Directory /var/www/catalog/catalog>
	                Require all granted
	        </Directory>
	        Alias /static /var/www/catalog/catalog/static
	        <Directory /var/www/catalog/catalog/static>
	                Require all granted
	        </Directory>
	        <DirectoryMatch "^/.*/\.git/">
	                Require all denied
	        </DirectoryMatch>
	        ErrorLog ${APACHE_LOG_DIR}/error.log
	        LogLevel info
	        CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>	
	```

	Source: 

	[https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/](https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/)

	[https://www.linux.com/learn/apache-ubuntu-linux-beginners](https://www.linux.com/learn/apache-ubuntu-linux-beginners)


#### Install and configure PostgreSQL.

- Install postgres

	`sudo apt-get install postgresql`

- Connect as root.

	```
	sudo su - postgres
	psql
	```

- Create a new database user named 'catalog' with the password 'catalog'

		`CREATE USER catalog PASSWORD 'catalog';`

- Create a new database owned by the catalog user.

	```
	ALTER USER catalog CREATEDB;
	CREATE DATABASE catalog WITH OWNER catalog;
	```

- Connect to the database and limit the permissions so that only the catalog user can create tables.

	```
	\c catalog
	REVOKE ALL ON SCHEMA public FROM public;
	GRANT ALL ON SCHEMA public TO catalog;
	```

- Quit psql and exit Postgres.

	```
	\q
	exit
	```

- Edit the create_engine line in mod_db/connect_db.py, db_setup.py, homework_items.py as follows.

	`create_engine('postgresql://catalog:catalog@localhost/catalog')`


- Check no remote connections are allowed.  

	* Open pg_bha.conf.  

		`sudo nano /etc/postgresql/9.3/main/pg_hba.conf`

	* Check the configuration.  
		```
		local   all             postgres                                peer
		local   all             all                                     peer
		host    all             all             127.0.0.1/32            md5
		host    all             all             ::1/128                 md5
		```


- Run `python db_setup.py`

	Sources:

	[https://wiki.postgresql.org/wiki/First_steps](https://wiki.postgresql.org/wiki/First_steps)

	[https://www.postgresql.org/docs/8.0/static/sql-alteruser.html](https://www.postgresql.org/docs/8.0/static/sql-alteruser.html)
	
	[https://www.digitalocean.com/community/tutorials/](https://www.digitalocean.com/community/tutorials/)
	
	[how-to-secure-postgresql-on-an-ubuntu-vps](how-to-secure-postgresql-on-an-ubuntu-vps)

#### Oauth Setup


- Go to Facebook developer dashboard and update OAuth authorized JavaScript origins to http://ec2-34-214-237-115.us-west-2.compute.amazonaws.com/, and modify redirect URIs

- Edit app\_id and app\_secret in fb\_client_secrets.json

- Go to Google developer dashboard and update OAuth authorized JavaScript origins to http://ec2-34-214-237-115.us-west-2.compute.amazonaws.com/ and redirect URIs, download the json, and copy the content into g\_client\_secrets.json.


- Update all the path to g\_client\_secrets.json and fb\_client\_secrets.json in the mod_auth/autho.py program as shown below.

	```
	CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/g_client_secrets.json', 'r').read())['web']['client_id']
    ```


#### Enable the catalog vitrual host


- Enable the virtual host
	
	`sudo a2ensite catalog`

- Activate the new configuration by running

 	`service apache2 reload`

	Or

- Restart Apache service

	`sudo service apache2 restart`


- Visit the IP address in a browser.

	* If you get Internal Server Error (500), check the error log.

		`sudo tail -f /var/log/apache2/error.log`


### Changes to \_init\__.py

- Fix for 404 Error 

	I moved the lines to register bluerpints out of the if block as the catalog module gets imported by the catatlog.conf file. 

	```
	from flask import Flask
	from mod_auth.auth import auth
	from mod_views.views import views
	from mod_api.api import api
	from mod_db.connect_db import db

	app = Flask(__name__)

	app.register_blueprint(auth)
	app.register_blueprint(views)
	app.register_blueprint(api)
	app.register_blueprint(db)

	if __name__ == '__main__':
	        app.secret_key = 'super_secret_key'
	        app.debug = True
	        app.run()
	```


