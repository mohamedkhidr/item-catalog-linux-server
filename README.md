# Linux server configuration  (Udacity)

### Description 
This is  linux server created on ubuntu linux server configured to serve the item catalog  web application which is written in python.
the server is fully secured and can be accessed remotely from any where 
- IP Address :  13.90.249.201
- SSH port :  2200
- Application URL : http://itemcatalog.eastus.cloudapp.azure.com


### Azure cloud service :
Create an account on microsoft cloud service (Azure) https://azure.microsoft.com/en-us/
azure service provide you a free service if you are a student.
i have created an ubuntu vm using this video https://www.youtube.com/watch?v=2qBvR-DMky4

### Softwares installed and configuration changes made
##### 1.  Create new user named grader and give it the permission to sudo
- SSH into the server through ``` ssh khidr@13.90.249.201 ```
- Run ``` sudo adduser grader ``` to create a new user named grader
- Create a new file in the sudoers directory with ``` sudo nano /etc/sudoers.d/grader ```
- Add the following text grader ALL=(ALL) NOPASSWD:ALL

##### 2. Update all  installed packages
 - ``` sudo apt-get update ```
 
 - ``` sudo apt-get upgrade ```
 
##### 3. Change SSH port from 22 to 2200 and configure ufw
 - ``` sudo ufw allow ssh ```
 - ``` sudo ufw allow www ```
 - ``` sudo ufw allow ntp ```
 - ``` sudo ufw allow 2200/tcp ```
 - ``` sudo ufw enable ```
 - Run  ``` sudo nano /etc/ssh/sshd_config ```
 - Change the port from 22 to 2200
 - ``` sudo ufw deny 22/tcp ``` 
 - Confirm by running ssh grader@13.90.249.201 -p 2200 
 
##### 4. Configure the local timezone to UTC
 - Run ```sudo dpkg-reconfigure tzdata```
 -  choose None of the above 
 - then choose UTC
 
##### 5. Configure key-based authentication for grader user
- in the home directory Run ``` sudo mkdir .ssh ```
- Run ``` sudo touch .ssh/authorized_keys ```
- Run ``` chmod 700 .ssh``` and ``` chmod 644 .ssh/authorized_keys ``` to protect them  
- in your local machine use command ``` ssh-keygen ``` to create rsa pairs 
- copy the public key and paste it in authorized_keys file using ``` sudo nano .ssh/authorized_keys ```
- Disable ssh login for root user
- Run ``` sudo nano /etc/ssh/sshd_config ```
- Change " PermitRootLogin  prohibit-password " line to PermitRootLogin no
- Disable password authentication 
- in line PasswordAuthentication change yes to no 
- Restart ssh with ``` sudo service ssh restart ```
- now all login changes are done 

##### 6. Install  Apache server with its python module
- Run ``` sudo apt-get install apache2 ```
- Run ``` sudo apt-get install python-setuptools libapache2-mod-wsgi ``` apache module
- Restart Apache ``` sudo service apache2 restart ```


 ##### 7. Install and configure PostgreSQL
- Install PostgreSQL ``` sudo apt-get install postgresql ```
- Check if no remote connections are allowed   
``` sudo  vim/etc/postgresql/10/main/pg_hba.conf  ```

- Login as user postgres  ``` sudo su - postgres ```

- Get into postgreSQL shell psql

-  Create a new database named catalog and create a new user named khidr 

-  ``` CREATE DATABASE catalog; ```
- ```  CREATE USER khidr; ```


- ``` ALTER ROLE khidr WITH PASSWORD 'password'; ``` to set the password


- ``` GRANT ALL PRIVILEGES ON DATABASE catalog TO khidr; ``` to give the user a permission on the database

##### 8 .Install git, clone and setup itemCatalog App project.
- Install Git using ``` sudo apt-get install git ```
- Run ``` cd /var/www ``` to go to the directory at which to put the app
 - Create the application directory ``` sudo mkdir catalog ```
- Move inside this directory using ``` cd catalog ```
- Clone the itemCatalog App using  ``` git clone https://github.com/mohamedkhidr/itemCatalog.git ```
- in files database_setup.py , fill_db.py and app.py 
change :
every ``` sqlite:///itemcatalog.db ```  to  ``` postgresql://khidr:password@localhost/catalog ```
and 
every ```client_secrets.json ```  to  ``` /var/www/catalog/itemCatalog/client_secrets.json ```
- Install pip ``` sudo apt-get install python-pip ```
- Use pip to install dependencies ``` sudo pip install -r requirements.txt ```
- Install psycopg2 ``` sudo apt-get -qqy install postgresql python-psycopg2 ```
- Run ``` sudo python database_setup.py ``` to create db schema
- Run ``` sudo python fill_db.py ``` to fill the database
- Create catalog.conf to edit: ``` sudo nano /etc/apache2/sites-available/catalog.conf ```

Add the following lines of code to the file to configure the virtual host.
``` 
<VirtualHost *:80>
	ServerName catalog
	ServerAdmin midokhidr4@gmail.com
	WSGIScriptAlias / /var/www/catalog/itemCatalog/run.wsgi
	<Directory /var/www/catalog/itemCatalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/catalog/itemCatalog/static
	<Directory /var/www/catalog/itemCatalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

- Enable the virtual host with the following command: ``` sudo a2ensite catalog ```
- Disable the default host with the following command: ``` sudo a2dissite default ```

- Create the .wsgi File
- Create the .wsgi File under /var/www/catalog/itemCatalog/:

 - Run ``` cd /var/www/catalog/itemCatalog ```
- ``` sudo nano run.wsgi  ```
Add the following lines of code to the run.wsgi file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/itemCatalog/")

from app import app as application
application.secret_key = 'Add your secret key'
```
- Restart Apache ``` sudo service apache2 restart ```


#### Author : mohamed khidr





