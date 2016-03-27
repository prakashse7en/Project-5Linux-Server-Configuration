P5: Linux Server Configuration
1) What is it?
In this project Travel Seven web application developed earlier for 
Project 3: Item Catalog will be hosted on a virtual server. Udacity and 
Amazon have provided a virtual server in Amazon’s Elastic Compute Cloud (EC2). 
Public Ip for launching this application is http://ec2-52-27-139-62.us-west-2.compute.amazonaws.com/

Steps:
-------
	*) goto udacity dev environment @ https://www.udacity.com/account#!/development_environment and 
	   download the private key and  move it to  .ssh folder
				command used is - "mv ~/Downloads/udacity_key.rsa ~/.ssh/"
	*)open terminal and type in
				command used is - "chmod 600 ~/.ssh/udacity_key.rsa"
	*)In terminal, type in
				command used is - "ssh -i ~/.ssh/udacity_key.rsa root@52.27.139.62"
	Now you are connected to Amazon EC2 machine in your terminal as "root"

1)New user creation - 
	reference - https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
	i) Create new user
		*)  $ adduser grader
	ii)Provide permission for the new user
		*) open sudo configuration
			$ visudo
		*) add new user below root 
			grader ALL=(ALL:ALL) ALL
		*)update and upgrade the installed packages
			$ sudo apt-get update
			$ sudo apt-get upgrade
2) Configure the local timezone to UTC
	  reference - https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
	i)Open the timezone selection dialog
		*)$ sudo dpkg-reconfigure tzdata ,choose 'None of the above' and choose UTC
3)Change the SSH port from 22 to 2200
	i)Change ssh config file:
		*) $ sudo nano /etc/ssh/sshd_config , change PORT from 22 to 2200
		*)change PermitRootLogin from without-password to no
		*)Restart SSH Service "$ /etc/init.d/ssh restart or # service sshd restart"
	ii)Create SSH keys
		reference - https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
					https://discussions.udacity.com/t/cannot-access-the-key-gen-created-for-linuxcourse/46105
					http://askubuntu.com/questions/727880/cannot-access-the-key-generated-pub-file
		*)generate keys in local machine "$ ssh-keygen" which creates both public and private key
		*)create a new folder .ssh in new user "grader"
			$ mkdir .ssh
			$ touch .ssh/authorized_keys this file will store all the public keys
			*)get the key content from  local machine(windows) "cat .ssh/linuxGrader.pub"
			this file is used for authentication with one key per line
			*)paste it in .ssh/authorized_keys file (Note: check if there is /n in your copied lines else you cant able to login again)
                                                     -----
			*)set file permission using chmod 700 .ssh,chmod 644 .ssh
			*login using the following command $ ssh -v grader@PUBLIC-IP -p 2200
4)Configure the Uncomplicated Firewall (UFW)
	i)Set firewall
		reference - https://help.ubuntu.com/community/UFW
		*)$ sudo ufw enable to enable firewall
		*)allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
			"$ sudo ufw allow 2200/tcp",
			"$ sudo ufw allow 80/tcp",
			"$ sudo ufw allow 123/udp"
5)Install and configure Apache to serve a Python mod_wsgi application 
	i)Install Apache
		reference - https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu
		*)$ sudo apt-get install apache2 check if its working fine by hitting your ip address in browser. Apache welcome page should be shown
	ii)Install mod_wsgi 
		reference - https://www.digitalocean.com/community/tutorials/installing-mod_wsgi-on-ubuntu-12-04
		*)$ sudo apt-get install python-setuptools libapache2-mod-wsgi
		*$ sudo service apache2 restart
6)Install git and clone and setup project project
	i)Install git 
		reference -https://help.github.com/articles/set-up-git/#platform-linux
		*) $ sudo apt-get install git
		*) $ git config --global user.name "YOUR NAME" 
		    $ git config --global user.email "YOUR EMAIL ADDRESS" set name and email address
	ii) Install Flask
		reference - https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
		*)$ sudo apt-get install libapache2-mod-wsgi python-dev
		*)enable mod_wsgi 
			$ sudo a2enmod wsgi
		*)Create flask app
			inside /var/www create directory 
					.)$ sudo mkdir catalog
					.)$ cd catalog and $ sudo mkdir catalog
					.)$ cd catalog and $ sudo mkdir static templates
					.)$ sudo nano __init__.py  this file contains the flask logic
		*)"$ sudo apt-get install python-pip" install pip for installing all python modules easily
		*)"$ sudo pip install virtualenv" installs virtual environment
		*)"$ sudo virtualenv venv" create a new virtual environment
		*)enable all permissions for virtual environment
			"$ sudo chmod -R 777 venv"
		*)"$ pip install Flask" installs flask
	iii)create a virtual host config file 
		*)$ sudo nano /etc/apache2/sites-available/catalog.conf
		*)paste the below lines inside your code 
			    <VirtualHost *:80>
				  ServerName PUBLIC-IP
				  ServerAlia AMAZON EC2LINK WITHOUT HTTP
				  ServerAdmin admin@PUBLIC-IP
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
			replace PUBLIC-IP with your ip and enable virtual host "$ sudo a2ensite catalog"
			replace AMAZON EC2LINK WITHOUT HTTP with ec2-52-27-139-62.us-west-2.compute.amazonaws.com 
			(Note no http or / at end else it cause error)
		*)"$ cd /var/www/catalog" and "$ sudo nano catalog.wsgi" and paste the below lines
			  #!/usr/bin/python
			  import sys
			  import logging
			  logging.basicConfig(stream=sys.stderr)
			  sys.path.insert(0,"/var/www/catalog/")

			  from catalog import app as application
			  application.secret_key = 'Add your secret key'
		*)Restart Apache - "$ sudo service apache2 restart"
	iv)Clone github 
		*)$ git clone https://github.com/prakashse7en/Project3_ItemCatalog_TravelSeven.git
		*)move all contents of Project3_ItemCatalog_TravelSeven to  "/var/www/catalog/catalog/"
		*)Make the repository inaccesible
			"$ cd /var/www/catalog/" and "$ sudo nano .htaccess" and paste "RedirectMatch 404 /\.git"
	v)Install other packages 
		*)$ source venv/bin/activate
		*)$ pip install httplib2
		*)$ pip install requests
		*)$ sudo pip install --upgrade oauth2client
		*)$ sudo pip install sqlalchemy
		*)$ sudo apt-get install python-psycopg2
5)Install PostgreSQL
	i)Installing and setting up database
		reference - https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
		*)$ sudo apt-get install postgresql postgresql-contrib
		*)replace the lines "engine = create_engine('sqlite:///travelseven.db')" with
			"engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')" in all the files where it is used
		*)Add new user for psql
			"$ sudo adduser catalog"
		*)switch to postgres user
			"$ sudo su - postgre" and "$ psql" is used to connect to the system
		*)Add catalog user for db operations
			"CREATE USER catalog WITH PASSWORD 'PASSWORD';"
			"ALTER USER catalog CREATEDB;"
		*)Create database 
			"CREATE DATABASE catalog WITH OWNER catalog;" and connect to database "\c catalog"
		*)Revoke all rights 
			"REVOKE ALL ON SCHEMA public FROM public;"
		*)Allow access only to catalog
			"GRANT ALL ON SCHEMA public TO catalog;"
		*)set up tables
			"$ python database_setup.py"
6)Add endpoints to FB and Google
	i)For fb
		*) login to https://developers.facebook.com/  go to settings your app and click settings from left hand side set of settings
			click advance tab and in Valid OAuth redirect URIs
			add your application ip in it  http://ec2-52-27-139-62.us-west-2.compute.amazonaws.com/
	ii)For gmail
		*)login to https://console.developers.google.com/project and select your project
		*)Navigate to APIs & auth > Credentials > Edit Settings and add your app in Authorized JavaScript origins
7)When amazon ip() is not working please refer to this link  https://discussions.udacity.com/t/application-shows-with-my-public-ip-address-but-not-with-my-amazon-ec2-instance-public-url/37117/7

			
Contacts
--------

o please send your queries regarding this project to prakash.dec20@gmail.com 
			
		


Notes to Reviewer
-----------------
udacity_key.rsa is 
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAuf88AwfwLe4Ld1jKHw2a7KdgsGXcIlcj9xZ+xSmgoOk6osx4
9dDRJBqEDoqlzGZD+b+sz8EKi6Vn97eMebZ+Bx+h+YR8f6wjuIzSUbMWXw0gJsQD
iXt8p2og7BuVbHCPOjPbUA0AEkl5amo28SHpGXe8NH8Apyyck5i3CfgwC0IATV+B
a2b6Xpg7XkPlswn1BS10+w9+pHgMjk0iaUp2wCCfvCh8P+zThVOM0ndL5fYaODNv
TfTXT+4ySaceWe06d8ZKuSb7uKmC1o5/A2RsKc56NVhWylfGbgVUhezTRllsctDM
2CiL/VjxTjYfjeEekVUYa7wvbS4o7+znkFZRtwIDAQABAoIBAQCT1ABmiFCksKX8
XV2IANA5d26pxMuJn6i+Ierv2X4JZlVsPweEmEshXtHGnPvZ0Q4F2goHtW4kP3q6
r+++bQUNtF6QanRpJO/fJk2jEaueMFh1dyU4iCUzCm7QObwxS+UKZVzR6wM7hZoy
seipDkKuMzQqpSZnuFVaGe6gxdmpl5i2Bs5ftjPhMWtlAh4A7lEuBNoEjk42YtkS
9Mms75yv75Sa3enNX5dE5oWcyZIaZSC9LcA010W57yOPB29KJHaj2GDH/67hoFq9
U5soxiUbjtqNc0PkJuj+coJc4nUUgYAPZKIfdzasMy3bDWGoNiS+0Ps3j/A8r0q+
4M/UMk7BAoGBANvQihf3jikdkEBIQcCqeS3566/qiIYtqCXGKlv1EqwxLqAWcgyV
BbcOKagWtEzedZLuVLkN7+nojNYZtNP/ELPC/JuZgC5l921Z3/HeIexd0qlpmycf
Wzlqb83BdQgPv4zi49CLWLJfgMbTTaotbLeMiRcuE1N1cNL1KcgHHRVFAoGBANid
jHSMKY8fvfYtztvtGaw1Lf4bsSUvXa9ktf+nPxivhN5fSBYz61/BhmWF8HQPSnNh
3IXgSERAZGZhMoFQWFXA8SL9/jYtizdZ7stxc6g/KojmuNC3tjmOAX1pqKgRacPM
nBnOynj+yy5NetEJ/scWdj38D8Fx0xTgt1WI0eTLAoGBALreJCeP2pj1ewZK5ysF
QZNmXYjllz6KXeIO/z/BrigYf4y0yCwOHBeswJkXBBw9GjLYzcmsIYL2oZP5spJu
yiIn51vYOPI42QlrWEhkEO7CLC69iprNu12qMHX4uqcpzCvXTtihPbwWGIHubJ35
k+zOWlUMZH2U319X8DcOZRkJAoGANql7OiXsjtt5ulfQ7Zqlcdlxo8AlMbcEMzB8
5Oi1eWtBYkQ1ErVDXkSdv5zPEtqQ9RDq7zWrxt1g+Jzqe8tkny8zKpthvRY9HViq
c9hLUVevSiC+3pyddWSqZ5V0JAVQ5UIK2lBes63IZATVS070ZhT6/aVP7Ibmt0wF
t6XxIYUCgYAOuhWuetAcVSHxgIWLiZy3X+PTY5DRuhSNS6k3wVwoNk/ibiz0tysB
fxaW+CmLNJrNcTK/LQf4DvJ/Y1GKPnOqSzmZ/TJiWVsgu8mK48h1fn1CUMLJ5qEh
7h1jmsS4mvbkYuJdWk3ca9gRn4mK9Ro5Z7flhY3MOrgQvLn7ImPN3w==
-----END RSA PRIVATE KEY-----


