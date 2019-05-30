# Linux Server Configuration

## Project Description

This is a project made for the completion of the Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

## Instructions

Take a baseline installation of a Linux server and prepare it to host a web applications. Secure the server from a number of attack vectors, install and configure a database server, and deploy an existing web applications onto it.

## Project info

- Public IP address: 18.232.104.248;
- URL: [http://18.232.104.248/](http://18.232.104.248/) and [http://ec2-18-232-104-248.compute-1.amazonaws.com/](http://ec2-18-232-104-248.compute-1.amazonaws.com/)
- Port: 2200
- Software/ Features used:
  - [Amazon Lightsail](https://amazonlightsail.com/) to create a Linux server instance.
  - The web application [Color-Catalog](https://github.com/Sonjya00/Color-Catalog), which is hosted on the server.
  - [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and [Vagrant](https://www.vagrantup.com/downloads.html) to work locally.
  - [Python](https://www.python.org/download/releases/3.0/) (latest version).
  - [PostgreSQL](https://www.postgresql.org/) as database server.
  - [Apache2](http://httpd.apache.org/)
  - [Google OAuth](https://console.developers.google.com/apis).

## How to login/logout (for graders)

- To login, type on the terminal `ssh -i ~/.ssh/grader_key -p 2200 grader@18.232.104.248`
- To logout, input `sudo shutdown -r now`

## Configurations made

### 1. Get a server

#### Create the server

- Login into Amazon Lightsail using an Amazon Web Services account. If you don't have an AWS account, create one.
- Click on "Create an instance".
- Choose Linux/Unix platform, "OS Only" and "Ubuntu 16.04 LTS".
- Choose an instance plan (the lower tier is fine).
- Give your instance a name.
- Click on "Create" and wait for it to start up.

#### SSH into the server

- At this point, the instance can be accessed via the website (by clicking on "Connect using SSH").
- To access it via the local terminal:
  - Go to the Account page on the Amazon Lightsail website, and select the "SSH keys" tab.
  - Download the keys (default is pre-selected).
  - Rename the downloaded file `lightsail_key.rsa`, and move in the local folder `~/.ssh` (if there isn't, cd in the home directory and create the directory by inputing `mkdir .ssh`)
  - Type in the terminal `chmod 600 ~/.ssh/lightsail_key.rsa` to fix the file permissions.
  - Now the instance can be accessed by inputing in the terminal the command `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.232.104.248` (for now the default port used is 22).

### 2. Secure the server

#### Update and upgrade installed packages

- To update/upgrade installed packages:
  - Input the commands `sudo apt-get update`,
  - Input `sudo apt-get upgrade`.
  - Input `sudo apt-get dist-upgrade`
- To enable automatic updates:
  - install the [unattended-upgrades](https://help.ubuntu.com/lts/serverguide/automatic-updates.html) package with `sudo apt install unattended-upgrades`
  - Input `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`, and uncomment the line `"${distro_id}:${distro_codename}-updates";`, then save the file.
  - Input `sudo nano /etc/apt/apt.conf.d/20auto-upgrades`, and change the file so that it is as follows:
    `APT::Periodic::Update-Package-Lists "1"; APT::Periodic::Download-Upgradeable-Packages "1"; APT::Periodic::AutocleanInterval "7"; APT::Periodic::Unattended-Upgrade "1";`

#### Configure the ports

- First, change the port configuration from the terminal.
  - On the terminal, input `sudo nano /etc/ssh/sshd_config`. Change the port number from 22 to 2200, then save the file.
  - Restart SSH with `sudo service ssh restart`.
  - To configure the ports permissions, input the following commands:
    - `sudo ufw default deny incoming`
    - `sudo ufw default allow outgoing`
    - `sudo ufw allow 2200/tcp`
    - `sudo ufw allow 80/tcp`
    - `sudo ufw allow 123/tcp`
    - `sudo ufw deny 22`
  - Enable the firewall by inputing `sudo ufw enable`
- Lastly, update the configuration on the Amazon Lightsail website too by selecting the tab "Networking" from the instance page. The configuration should looks as follow:
  - HTTP - TCP - 80
  - Custom - UDP - 123
  - Custom - TCP - 2200

### 3. Give the grader access

#### Add the grader as sudoer

- Add the grader user by inputing on the terminal `sudo adduser grader`.
- Set the password by inputing it twice.
- To give sudo privileges to grader, create the file `/etc/sudoers.d/grader`.
- Input `sudo nano /etc/sudoers.d/grader` and change the content of the file to: `grader ALL=(ALL:ALL) ALL`

#### Add SSH key for the grader

- On the local machine:
  - generate the key pair by using `ssh-keygen`.
  - Enter file in which to save the key (I called the file "grader_key"), then move it to the local `~/.ssh/` directory.
  - Enter a passphrase twice.
  - This generates 2 files: `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`.
  - Input `cat .ssh/grader_key.pub` to read its content. Copy the content. This will be used on the next step.
- On the server, logged as grader:
  - Create a new ssh directory in the home directory with `mkdir .ssh`.
  - Input `touch ssh/authorized_keys` to create the file.
  - Input `nano .ssh/authorized_keys` and paste the content previously copied from the local machine.
  - Fix the permissions of the .ssh directory and the ssh key file by inputing `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
  - Check if the password based login is disabled by inputing `sudo nano /etc/ssh/sshd_config`. If there is written `PasswordAuthentication yes`, change it to `PasswordAuthentication no`, then save the file.
  - Restart the service by inputing `sudo service ssh restart`.
- Now it is possible to access the instance as grader with `ssh -i ~/.ssh/grader_key -p 2200 grader@18.232.104.248`.

### 4. Further configuration

#### Configure the local timezone to UTC

- Configure the timezone with the command `sudo dpkg-reconfigure tzdata`
- Select "None of the above", then "UTC".

#### Install Apache

- To install Apache, input `sudo apt-get install apache2`
- Open http://18.232.104.248/ in the browser. If you see the Apache2 Ubuntu Default Page, this means that Apache is working correctly.
- Install mod_wsgi with `sudo apt-get install libapache2-mod-wsgi`.
- Enable mod_wsgi with the command `sudo a2enmod wsgi`.

#### Install PostgreSQL

- To install PostgreSQL, type `sudo apt-get install postgresql`.
- Make sure PostgreSQL is not allowing remote connections by checking the file `/etc/postgresql/9.5/main/pg_hba.conf`.
- Switch to PostgreSQL default user by inputing `sudo su - postgres`.
- Open PostgreSQL with `psql`.
- Create the user "catalog" with `CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog'`
- Give it the ability to create database tables with `ALTER ROLE catalog CREATEDB`.
- Exit psql with the command `\q`.
- Switch out from the "postgres" user and into the "grader" user with `exit`.

#### Create new catalog user

- Create a new Linux user named "catalog" with the command `sudo adduser catalog`. Set the password by inputing it twice.
- To give the new catalog user sudo permissions, input `sudo visudo`, and add the line `catalog ALL=(ALL:ALL) ALL` under `root ALL=(ALL:ALL) ALL`, then save the file.
- Enter the catalog user with `su - catalog`.
- Create a database with `createdb catalog`.
- Switch out from catalog user and into grader user with `exit`.

#### Install git

- To install git, run `sudo apt-get install git`.

### 5. Deploy the project

#### Clone project

- While logged as grader, create a new directory: `mkdir /var/www/catalog`.
- Cd into the new directory and then clone the project there: `sudo git glone https://github.com/Sonjya00/Color-Catalog.git catalog`.
- While on `/var/www/`, change the ownership of the catalog directory with `sudo chown -R grader:grader catalog/`.

#### Create a new client_secrets.json file (Google OAuth)

- Visit https://www.hcidata.info/host2ip.cgi
- Input the IP address and get host name (https://www.hcidata.info/host2ip.cgi)
- Go to the Google API Console.
- From the left menu, select "API and Services", then "Credentials".
- Click on the button "Create credentials", then select OAuth client ID" from the dropdown menu.
- Add the Amazon Lightsail instance's public IP and the host name just found as Authorized JavaScript origins, then save.
- Download the JSON file.
- Open it and copy its content.
- On the terminal, while logged as grader, cd into `/var/www/catalog/catalog/`.
- Create a new JSON file with `touch client_secrets.json`.
- Open the new file with `nano client_secrets.json`, and paste the previously copied content, then save the file.

#### Make changes to the original project files

- While logged as grader, cd into the project's repo. Rename the file `views.py` as `__init__.py`.
- Enter `sudo nano __init__.py` to apply some needed changes to that file.
- Update line 19 as follows: `CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r'$ 'web']['client_id']`.
- Change line 22 as follows: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`.
- Change the last line as follows: `app.run()`.
- Enter `sudo nano models.py` and change line 63 as follows: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`.
- Enter `sudo nano addcolors.py` and change line 6 as follows: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`.

#### Install the virtual environment

- Install the virtual environment with `sudo apt-get install python-virtualenv`
- Cd into `/var/www/catalog/catalog/`.
- Create the virtual environment with `sudo virtualenv -p python3 venv3`.
- Change the ownership to grader with `sudo chown -R grader:grader venv3`.
- Activate the new environment with `venv3/bin/activate`.

#### Install dependencies

- Install the following:
  - `sudo apt-get install python-psycopg2`
  - `sudo apt-get install libpq-dev`
  - `pip3 install sqlalchemy`
  - `pip3 install psycopg2`
  - `pip3 install --upgrade oauth2client`
  - `pip3 install flask`
  - `pip3 install flask_httpauth`
  - `pip3 install httplib2`

-Deactivate the virtual machine with `deactivate`.

#### Populate database

- Run `python3 models.py`, then `python3 addcolors.py`.

#### Configure .conf file

- Create the file `sudo touch /etc/apache2/sites-available/catalog.conf`.
- Add the following lines to the file.

  `<VirtualHost *:80>`
  `ServerName 18.232.104.248`
  `ServeAlias ec2-18-232-104-248.compute-1.amazonaws.com/login`
  `WSGIScriptAlias / /var/www/catalog/catalog.wsgi`
  `<Directory /var/www/catalog/catalog/>`
  `Order allow,deny`
  `Allow from all`
  `</Directory>`
  `Alias /static /var/www/catalog/catalog/static`
  `<Directory /var/www/catalog/catalog/static/>`
  `Order allow,deny`
  `Allow from all`
  `</Directory>`
  `ErrorLog ${APACHE_LOG_DIR}/error.log`
  `LogLevel warn`
  `CustomLog ${APACHE_LOG_DIR}/access.log combined`
  `</VirtualHost>`

- Enable the virtual host with the command `sudo a2ensite catalog`.
- Reload Apache with `sudo service apache2 reload`.

#### Configure .wsgi file

- Create new file with `sudo touch /var/www/catalog/catalog.wsgi
- Add the following content to the file:

  `#!/usr/bin/python`
  `import sys`
  `import logging`
  `logging.basicConfig(stream=sys.stderr)`
  `sys.path.insert(0,"/var/www/catalog/")`

  `from catalog import app as application`
  `application.secret_key = 'super_secret_key'`

- Reload Apache with `sudo service apache2 reload`.

#### Disable apache2 default page

- Disable the default Apache site with the command `sudo a2dissite 000-default.conf`.
- Reload Apache with `sudo service apache2 reload`.

## Resources

- [https://www.udacity.com/course/configuring-linux-web-servers--ud299](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
- [https://help.ubuntu.com/community/SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)
- [https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac](https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac)
- [https://help.ubuntu.com/lts/serverguide/automatic-updates.html](https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac)
- [https://help.ubuntu.com/community/UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- [https://help.ubuntu.com/lts/serverguide/postgresql.html.en](https://help.ubuntu.com/lts/serverguide/postgresql.html.en)
- [https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [https://www.hcidata.info/host2ip.cgi](https://www.hcidata.info/host2ip.cgi)
- [http://httpd.apache.org/docs/current/configuring.html](http://httpd.apache.org/docs/current/configuring.html)
