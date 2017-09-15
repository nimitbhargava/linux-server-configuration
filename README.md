# linux-server-configuration


```
sudo apt-get update
sudo apt-get upgrade -y
```

#### Create a new user named `grader`

```
adduser grader
```

#### Give the user `grader` permission to sudo

1. Log into the remote VM as *ubuntu* user through ssh: `$ ssh -i KEY_FROM_AMAZON.pem ubuntu@13.126.174.25`.
2. Add a new user called *grader*: `$ sudo adduser grader`.
3. Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.

#### Set up SSH Authentication

Generate SSH key pairs, then copy the contents of the generated .pub file to the clipboard
```
# RUN ON LOCAL MACHINE
1. Generate an encryption key **on your local machine** with: `$ ssh-keygen -f ~/.ssh/grader.rsa`.
2. Log into the remote VM as *root* user through ssh and create the following file: `$ touch /home/grader/.ssh/authorized_keys`.
3. Copy the content of the *udacity_key.pub* file from your local machine to the */home/grader/.ssh/authorized_keys file you just created on the remote VM. Then change some permissions:
	1. `$ sudo chmod 700 /home/grader/.ssh`.
	2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	3. Finally change the owner from *root* to *grader*: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/grader.rsa grader@13.126.174.25`.
```

Configure public key on server.
   As the `grader` user paste `.pub` file contents in to `.ssh/authorized_key` file
```
# RUN ON SERVER
su grader
mkdir ~/.ssh
nano ~/.ssh/authorized_keys
```

Set correct permissions
```
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
```

#### Change the SSH port from 22 to 2200

Open SSH config file
```sh
vim /etc/ssh/sshd_config
```

Change `Port 22` to `Port 2200`


# IMP - `$ ssh -i ~/.ssh/grader.rsa -p 2200 grader@13.126.174.25` DONT WORK!

#### Remote login of the root user has been disabled

Open SSH config file

```sh
vim /etc/ssh/sshd_config
```

Ensure `PermitRootLogin` has a value  no`


#### Enforce SSH Authentication (i.e prevent password login)

Open SSH config file
```sh
vim /etc/ssh/sshd_config
```

Ensure `PasswordAuthentication` has a value  no`


#### Restart SSH service

```sh
sudo service ssh restart
```

#### Configure the Uncomplicated Firewall (UFW)

Block all incoming requests
```sh
sudo ufw default deny incoming
```

Allow all outgoing requests
```sh
sudo ufw default allow outgoing
```

Allowing incoming connections for SSH (port 2200)
```sh
sudo ufw allow 2200/tcp
```

Allowing incoming connections for HTTP (port 80)
```sh
sudo ufw allow www
```

Allowing incoming connections for NTP (port 123)
```sh
sudo ufw allow ntp
```

Enable `ufw`
```sh
sudo ufw enable
```

#### Configure the local timezone to UTC

Reconfiguring the tzdata package
```sh
sudo dpkg-reconfigure tzdata
# select `None of the above` then `UTC`
```

#### Install and configure Apache to serve a Python mod_wsgi application

Install required packages
```sh
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```
