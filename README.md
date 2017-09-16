# linux-server-configuration

### Steps to set up linux server and configure it

##### A. Sign up for [Amazon Lightsail](https://lightsail.aws.amazon.com) account.
##### B. Create a new [Amazon Lightsail instance](https://lightsail.aws.amazon.com/ls/webapp/create/instance).
Select OS only - choose Ubuntu 16.04 LTS
##### C. Adding TCP 2200 port in the firewall
1. Go to the instance
2. Click on Networking tab
3. Add a new custom firewall with Application -> Custom, Protocol -> TCP, and Port range -> 2200
##### D. Downloading SSH key
 1. [Download default SSH key pair](https://lightsail.aws.amazon.com/ls/webapp/account/keys) is downloaded as *LightsailDefaultPrivateKey-ap-south-1.pem*
 2. Store the key under ~/.ssh folder in your **LOCAL** machine.
##### E. SSHing to the Amazon Lightsail instance
1. Open terminal
2. Enter `$ ssh -i ~/.ssh/LightsailDefaultPrivateKey-ap-south-1.pem ubuntu@PUBLIC_IP` This will log you into the Amazon Lightsail instance.
##### F. Create a new user named grader and grant them sudo permissions
1. `ubuntu@ip-172-26-15-86:~$ sudo adduser grader`. Remember the password as you will need it later to login to grader user. 
2. `ubuntu@ip-172-26-15-86:~$ sudo nano /etc/sudoers.d/grader` -> Add `grader ALL=(ALL:ALL) ALL` and save
##### G. Generating and setting SSH keys for grader user
On you **LOCAL** machine terminal enter `ssh-keygen`. And save the public/private keys to ~/.ssh on local machine
##### H. Deploy SSH public key on Development Environment
1. `ubuntu@ip-172-26-15-86:~$ su - grader`
2. `grader@ip-172-26-15-86:~$ nano .ssh/authorized_keys` Copy the grader public key generated at ~/.ssh and copy the content of they public key to .ssh/authorized_keys
3. Change permissions `grader@ip-172-26-15-86:~$ chmod 700 .ssh; chmod 644 .ssh/authorized_keys`
4. Change owner to grader `grader@ip-172-26-15-86:~$ sudo chown -R grader:grader /home/grader/.ssh`
5. Restart the service `grader@ip-172-26-15-86:~$ sudo service ssh restart`
##### I. Change the SSH port from 22 to 2200
1. `ubuntu@ip-172-26-15-86:~$ sudo nano /etc/ssh/sshd_config` Modify Port 22 to Port 2200
2. Restart the service `ubuntu@ip-172-26-15-86:~$ sudo service ssh restart`
- Now you can access the ubuntu instance via `ssh -i ~/.ssh/LightsailDefaultPrivateKey-ap-south-1.pem -p 2200 ubuntu@13.126.75.183` from your local machine terminal. 
- Now you can access the grader instance via `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@13.126.75.183` from your local machine terminal. 
##### J. Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
```
Check if firewall is enabled by `sudo ufw status`
##### K. Configure the Development Environment (instance) timezone to UTC
Configure the time zone `ubuntu@ip-172-26-15-86:~$ sudo dpkg-reconfigure tzdata` -> None of these -> UTC
##### L. Disable Root login and Enforce key-based authentication
`ubuntu@ip-172-26-15-86:~$ sudo nano /etc/ssh/sshd_config` 
and modify to 
```
PermitRootLogin no
PasswordAuthentication no
```
##### M. Making the applications up-to-date
`ubuntu@ip-172-26-15-86:~$ sudo apt-get update` and then `ubuntu@ip-172-26-15-86:~$ sudo apt-get upgrade`
##### N. Install Apache2, mod_wsgi and git
```
ubuntu@ip-172-26-15-86:~$ sudo apt-get install apache2
ubuntu@ip-172-26-15-86:~$ sudo apt-get install apache2 libapache2-mod-wsgi git` and enable mod_wsgi `ubuntu@ip-172-26-15-86:~$ sudo a2enmod wsgi`
```
##### O. Install and Configure PostgreSQL
1. `ubuntu@ip-172-26-15-86:~$ sudo apt-get install postgresql postgresql-contrib libpq-dev python-dev`
2. Login as postgres User(Default user), and get into psql shell 
```
ubuntu@ip-172-26-15-86:~$ sudo su - postgres
postgres@ip-172-26-15-86:~$ psql
```
3. Create new User named *catalog*, a new Database *catdb*, and Connect to the Database
```
postgres=# CREATE USER catalog WITH PASSWORD 'password';
postgres=# CREATE DATABASE catdb WITH OWNER catalog;
postgres=# \c catdb
```
4. Revoke all rights from public, and allow only catalog to perform transactions
```
catdb=# REVOKE ALL ON SCHEMA public FROM public; GRANT ALL ON SCHEMA public TO catalog;
```
##### P. Install python app dependencies
```
ubuntu@ip-172-26-15-86:~$ sudo apt-get install python-pip
ubuntu@ip-172-26-15-86:~$ sudo pip install Flask==0.9
ubuntu@ip-172-26-15-86:~$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests
```
##### Q. Clone the application files
1. `ubuntu@ip-172-26-15-86:~$ cd /var/www/`
2. `ubuntu@ip-172-26-15-86:/var/www$ sudo git clone https://github.com/nimitbhargava/open-catalog.git project`
3. Create WSGI file
```
ubuntu@ip-172-26-15-86:/var/www$ cd project/
ubuntu@ip-172-26-15-86:/var/www/project$ sudo nano catalog.wsgi
```
Content for catalog.wsgi
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/project/")

from application import app as application
application.secret_key = 'ultra_secret_key'
```
##### R. Change the engine inside the .py files(`application.py`, `database_setup.py`, `lotsofcategoryanditem.py`) to `engine = create_engine('postgresql://catalog:password@localhost/catdb')`
##### S. Seed the database
```
ubuntu@ip-172-26-15-86:/var/www/project$ python database_setup.py
ubuntu@ip-172-26-15-86:/var/www/project$ python lotsofcategoryanditem.py
```

