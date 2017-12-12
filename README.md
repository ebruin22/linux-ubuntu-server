# Linux Server Configuration 

This project outlines the process to launch a linux server using ubuntu and Amazon Lightsail. 

## Server Details ##
Please visit http://18.216.230.178 for the website deployed using the steps outlined below.
This server uses ssh port 2200.

## Overview of Tasks ##
- [ ] Start a new Ubuntu Linux server instance in Amazon Lightsail
- [ ] Launch your new instance using the <connect using SSH> button
- [ ] Secure server by updating packages
- [ ] Configure UFW firewall and change SSH port from 22 to 2200
- [ ] Install VirtualBox and Vagrant in local machine (if not installed yet)
- [ ] Access instance via ssh in your virtual machine
- [ ] Create new user named grader and give user sudo access
- [ ] Create security key for user grader
- [ ] Change local timezone to UTC
- [ ] Install and configure Apache2 to serve python app
- [ ] Install and configure PostgreSQL 
- [ ] Create PostgreSQL user and empty database
- [ ] Install packages needed for your catalog app (see the imports in your python files)
- [ ] Clone and setup your catalog app
- [ ] Configure and enable a virtual host for the catalog app
- [ ] Create the .wsgi file

 Your're Done! 

### Start a new Ubuntu Linux server using Amazon Lightsail ###
1. Go to the Amazon Lightsail page
2. Register or log into your account
3. Click the create instance button
4. On the create an instance page, select platform as: Linux/Unix
5. Click on OS only button and choose Ubuntu from the options
6. Choose instance plan and name the instance
7. Click Create

### Update Server Packages ###
In the terminal that opened onced you connected via the Amazon Lightsail page, enter the following commands:
```
		sudo apt-get update
		sudo apt-get upgrade
```
You may need to type Y to install updates during the installation process

### Configure UFW firewall and change SSH port  ###
1. Confirm that the UFW firewall is inactive by typing:
```
		sudo ufw status 
```
2. Deny all incoming requests by default
```
		sudo ufw default deny incoming
```
3. Enable all outgoing requests:
```
sudo ufw default allow outgoing
```
4. Allow ports needed by server
```		
		sudo ufw allow ssh
		sudo ufw allow 2222/tcp
		sudo ufw allow wwww 
```
5. Enable ufw firewall
```
		sudo ufw enable
		sudo ufw status
```
 	Make sure firewall is active and ports enabled
6. Add port 2200 to your Amazon Lightsail instance, by going to the network tab and scrolling the firewall section. Add a new port by clicking "+ Add Another" choose "Custom" for application, TCP for protocol, and 2200 for port range and save changes.
7. Before changing ssh port make sure to the firewall is open to ssh port using "sudo ufw status" otherwise you may not be able to log into your instance 
8. Change the ssh port by typing the following commands:
```
		sudo vim /etc/ssh/sshd_config
```
9. Change change the line Port 22 to: Port 2200
10. Restart SSH service using command:
```		
		sudo service ssh restart
```
You will now need to log in using the command:
```
		ssh ubuntu@YOUR.IP.ADDRESS -i ~/.ssh/your_key_filename -p 2200```
Warning: You will need to download your default key first before logging into the instance from your terminal. See Access Instance Via SSH

### Launch Vagrant Virtual Machine ###
1. Dowload and install VirtualBox
2. Download and install Vagrant
3. Once both are installed, run the virtual machine with the following commands:
```
  		vagrant init ubuntu/trusty64
 		vagrant up 
 ```
### Access Instance Via SSH ###
1. Download your default private key for your Amazon lightsail instance
2. Open a new terminal window to create .ssh folder in your local home directory (not Vagrant) using command:
``` 
		mkdir ~/.ssh
```
3. Move the private key to your local .ssh folder using command (replace your_key_filename with your key's file name including extension):
```
		mv ~/Downloads/your_key_filename ~/.ssh/
```
4. Set permissions by entering the following command
```		
		chmod 600 ~/.ssh/your_key_filename
```
5. Access your instance via ssh with the following command: 
```
		ssh ubuntu@YOUR.IP.ADDRESS -i ~/.ssh/your_key_filename -p 2200
```

### Create new user ###
1. To create sudo (or super user access)  user and give sudo type the following command in the terminal:
		```- sudo adduser grader```
2. To allow user sudo access, the user will need to be added to the sudoers.d file, to do so type the following commands:
```
		vim /etc/sudoers
		touch /etc/sudoers.d/grader
		vim /etc/sudoers.d/grader
```
3. Type the following inside the editor for sudoers.d/grader and save the file
```
		grader ALL=(ALL:ALL) ALL
```

### Create SSH Key For New User ###
1. On local directory (not Vagrant), move to your .ssh directory using command:
```
		cd ~/.ssh
```
2. Create a new key pair using the command
```
		ssh-keygen
```
3. Save the key in your ~/.ssh directory on the local machine
4. In your local machine (not Vagrant), type the following command to get the key text (replace your key's filename including extension):
```
		cat .ssh/your_key_filename
```
5. Copy the text in the output as this is the public key
6. In your virtual machine, create directory to store public keys using the following:
``` 
		mkdir .ssh
		touch .ssh/authorized_keys
```
7. To deploy the public key for the new user, enter the following commands in your virtual machine:
```
		su - grader
		mkdir .ssh
		touch .ssh/authorized_keys
		vim .ssh/authorized_keys
```
8. Paste the public key text copied during step 5 of this category and save the file
9. Set permissions for the public key using the following commands:
```		
		chmod 700 .ssh
		chmod 644 .ssh/authorized_keys
```
10. Reload SSH to access server under new users by typing:
```
		service ssh restart
```
*** The new user will be able to log in using the following command (make sure to replace -->your_grader_key_filename<-- to your new user's key filename:
```
		ssh grader@YOUR.IP.ADDRESS -i ~/.ssh/your_grader_key_filename -p 2200
```

### Change Timezone To UTC ###
1. Configure the time zone 
```		
		sudo dpkg-reconfgure tzdata
```
2. Choose "None of the above" for Geographic area, then ok, choose "UTC" from option, then ok

### Install And Configure Apache2 ###
1. Install Apache package using command:
```
		sudo apt-get install apache2
```
2. Install and enable mod_wsgi
```
		sudo apt-get install libapache2-mod-wsgi
		sudo a2enmod wsgi
```
3. Restart Apache2 
```
		sudo service apache2 restart
 ```

### Install and configure PostgreSQL ###
1. Install PostgreSQL 
```
		sudo apt-get install postgresql 
```
2. Confirm that remote connections are not allowed using the following commands:
```
		sudo cd cat /etc/postgresql/YOUR_VERSION_NUMBER/main.pg_hba.conf
```

WARNING: You must replace YOUR_VARSION_NUMBER with your POSTGRESQL version number. You can check by running commands:
```
		cd /etc/postgresql/
		ls -al
```
	Note the folder with the number and use it in the command from step 2.

3. Check in the output from step 2, that connections are only allowed from local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6.
You may need to use command and make changes:
```
		sudo nano /etc/postgresql/YOUR_VERSION_NUMBER/main.pg_hba.conf
```

### Create PostgreSQL user And Empty Database ###
1. Login as user "postgres" using command
```
		sudo su - postgres
```
2. Enter the PostgreSQL shell using command 
```
		psql
```
3. Create a PostgreSQL user called catalog with the following command:
```
		CREATE USER catalog;
```
4. Create empty databse
```
		CREATE DATABASE catalog;
```
5. Set password for user catalog
```
		ALTER ROLE catalog WITH PASSWORD 'password';
```
	Replace 'password' with the unique user password you would like to use.
6. Grant permission to user catalog
```
		GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
7. Exit the postgreSQL shell
```
		\q
```
8. Logout of postgreSQL
```
		exit
```


### Install Packages Needed By Python App ###
1. Install python, psycopg2, flask, sqlalchemy, pip, oauth2client, requests, httplib2, seasurf, git using the following commands
``` 
		sudo apt-get install python-psycopg2 python-flask
		sudo apt-get install python-sqlalchemy python-pip
		sudo pip install oauth2client
		sudo pip install requests
		sudo pip install httplib2
		sudo pip install flask-seasurf
		sudo apt-get install git
```
### Clone and setup your catalog app ###
1. Move to directory /var/www/ using
```
		cd /var/www/
```
2. Create directory productCatalog
```
		sudo mkdir productCatalog
```
3. Move to new directory to clone project
```
		cd productCatalog
```
4. Clone from Github repository
```
		git clone YOUR_GITHUB_PROJECT_LINK
```
	Your Github repository will show the project link once you click the clone button in your github project webpage.
5. Rename your project using command
```
		sudo mv ./GITHUB_PROJECT_NAME ./
```
	Replace GITHUB_PROJECT_NAME with your unique github name and YOUR_NEW_PROJECT_NAME with the new name you want to give the app folder
6. Move to your new directory 
```
		cd YOUR_NEW_PROJECT_NAME
```
7. Rename the python file that runs your app to name ___init___.py 
```
		sudo mv YOUR_PYTHON_EXE_FILE.py __init__.py
```
	Replace YOUR_PYTHON_EXE_FILE with your project's main python filename. This is the python file that runs main and executes the application.
8. Configure python files that connect the app to the a database by replacing your current sqlalchemy create_engine line with the following
```
		engine = create_engine('postgresql://catalog:YOUR_POSTGRESQL_PASSWORD@localhost/catalog')
```
	Replace YOUR_POSTGRESQL_PASSWORD with the password you created for user catalog in step 5 of "Create PostgreSQL user And Empty Database"
9. Initialize the database from your python file
```
		sudo python database_setup.py
```

### Configure And Enable A Virtual Host ###
1. Create configuration file for your python app
```
		sudo nano /etc/apache2/sites-available/YOUR_NEW_PROJECT_NAME.conf
```
	Replace YOUR_NEW_PROJECT_NAME with the name of your project name in step 5 from "Clone and setup your catalog app"
2. Change the bold text of YOUR_NEW_PROJECT_NAME.conf file
```
			 <VirtualHost *:80>
			 	ServerName ***YOUR_AMAZON_AMAZON_LIGHTSAIL_IP***
				ServerAdmin admin@***YOUR_AMAZON_AMAZON_LIGHTSAIL_IP***
				WSGIScriptAlias / /var/www/***productCatalog***/***YOUR_NEW_PROJECT_NAME***.wsgi
				<Directory /var/www/***productCatalog***/***YOUR_NEW_PROJECT_NAME***/ >
					Order allow,deny
					Allow from all
				</Directory>
					Alias /static /var/www/***productCatalog***/***YOUR_NEW_PROJECT_NAME***/static
				<Directory /var/www/***productCatalog***/***YOUR_NEW_PROJECT_NAME***/static/>
					Order allow,deny
					Allow from all
				</Directory>
				ErrorLog ${APACHE_LOG_DIR}/error.log
				LogLevel warn
				CustomLog ${APACHE_LOG_DIR}/access.log combined
			</VirtualHost>
```
	Save changes
3. Enable the virtual host 
```
		sudo a2ensite YOUR_NEW_PROJECT_NAME
```


### Create the .wsgi File ###
1. Move the project file directory 
```
		cd /var/www/projectCatalog
```
2. Create wsgi file
	```sudo nano productCatalog.wsgi```
3. Add the following information to the wsgi file
```
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/productCatalog/")
		from productCatalog import app as application
		application.secret_key = 'Add your secret key'
```

4. Restart apache
```
		sudo service apache2 restart
```

### References ###
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
