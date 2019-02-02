# linux-server-configuration

Part of the udacity [Full Stack Web Developer Nanodegree.](https://udacity.com/course/full-stack-web-developer-nanodegree--nd004)

### Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your [existing web applications](https://github.com/AdeebTwait/linux-server-configuration) onto it.

## Software/Package Installed
- apache2
- libapache2
- mod
- wsgi
- PostgreSQL
- git
- pip
- virtualenv
- flask
- sqlalchemy
- finger
- Apache2
- mod_wsgi
- httplib2
- PythonRequests
- oauth2client
- libpq-dev
- Psycopg2
- Putty

## Set up server
- Go to [Microsoft Azure](https://azure.microsoft.com/en-us/free/students/) for students and make a free account using your university email (.edu).
- Go to this [link](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal) and follow the steps to create your ubuntu VM.

## SSH into the server 
- Go to this [article](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ssh-from-windows) and follow the steps to make a ssh connection to your ubuntu server that you've created in the step 1, (I used Putty software because of its GUI).

## Secure the server
### Update and upgrade installed packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
### Change the SSH port from 22 to 2200
- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from 22 to 2200.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: `sudo service ssh restart`.

### Configure the Uncomplicated Firewall (UFW)
- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw status                  # The UFW should be inactive.
sudo ufw default deny incoming   # Deny any incoming traffic.
sudo ufw default allow outgoing  # Enable outgoing traffic.
sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
sudo ufw allow www               # Allow HTTP traffic in.
sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
```
- Turn UFW on: `sudo ufw enable`. The output should be like this:
```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```
- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```
- Exit the SSH connection: `exit`.

- Open your virtual machine from azure dashboard and click on `Networking` tab, and then change the firewall configuration to match the internal firewall settings above. 

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

![1](https://user-images.githubusercontent.com/16962426/52150592-24b12c00-2679-11e9-868d-bda37c0bdedc.PNG)

- Now try to connect to your server with port `2200` using putty or command line.

## Use `Fail2Ban` to ban attackers
`Fail2Ban` is an intrusion prevention software framework that protects computer servers from brute-force attacks.

- Install Fail2Ban: `sudo apt-get install fail2ban`.
- Install sendmail for email notice: `sudo apt-get install sendmail iptables-persistent`.
- Create a copy of a file: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- Change the settings in `/etc/fail2ban/jail.local` file:
```
set bantime = 600
destemail = useremail@domain
action = %(action_mwl)s 
```
- Under [sshd] change `port = ssh` by `port = 2200`.
- Restart the service: `sudo service fail2ban restart`.

## Give `grader` access
### Create a new user account named `grader`
- While logged in, add user: `sudo adduser grader`.
- Enter a password (twice) and fill out information for this new user.
### Give `grader` the permission to sudo
- Edits the sudoers file: `sudo visudo`.

- Search for the line that looks like this:
```
root    ALL=(ALL:ALL) ALL
```
- Below this line, add a new line to give sudo privileges to grader user.
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
- Save and exit using CTRL+X and confirm with Y.

- Verify that grader has sudo permissions. Run `su - grader`, enter the password, run `sudo -l` and enter the password again. The output should be like this:

![2](https://user-images.githubusercontent.com/16962426/52151338-6b078a80-267b-11e9-95c1-21f91bd2f72e.PNG)

### Create an SSH key pair for grader using the `ssh-keygen` tool
#### - On the local machine:
- Run `ssh-keygen`
- Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`
- Enter in a passphrase twice. Two files will be generated ( `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
- Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
- Log in to the grader's virtual machine
#### - On the grader's virtual machine:
- Create a new directory called `~/.ssh` (mkdir .ssh)
- Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
- Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
- Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to no
- Restart SSH: `sudo service ssh restart`

- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@your-publicip`

## Prepare to deploy the project
### Configure the local timezone to UTC
- While logged in as grader, Check the timezone with the `date` command. This will display the current timezone after the time.
- If it's not UTC change it with this command: `sudo timedatectl set-timezone UTC`

### Install and configure Apache to serve a Python mod_wsgi application
- While logged in as grader, install Apache: `sudo apt-get install apache2`.
- Enter public IP of the Microsoft Azure instance into browser. If Apache is working, you should see:

<img width="599" alt="screen6" src="https://user-images.githubusercontent.com/16962426/52154482-b1aeb200-2686-11e9-9ece-e3572f7ed260.png">

- My project is built with Python 3. So, I need to install the Python 3 mod_wsgi package:
`sudo apt-get install libapache2-mod-wsgi-py3`.

- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.

### Install and configure PostgreSQL
- While logged in as grader, install PostgreSQL: `sudo apt-get install postgresql`.

- PostgreSQL should not allow remote connections. In the `/etc/postgresql/10/main/pg_hba.conf` file, you should see:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

- Switch to the postgres user: sudo su - postgres.
- Open PostgreSQL interactive terminal with psql.
- Create the `catalog` user with a password
```
postgres=# CREATE USER catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
postgres=# \c catalog;
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
```
- List the existing roles: `\du`. The output should be like this:
```
 List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 ```
 - Exit psql: `\q`.
 - Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
 ```
   List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```
- Exit psql: `\q`.

- Switch back to the grader user: `exit`.

### Install git
- While logged in as grader, install git: `sudo apt-get install git`.

## Deploy the Item Catalog project
### Clone and setup the Item Catalog project from the GitHub repository
- While logged in as `grader`, create `/var/www/catalog/` directory.
- Change to that directory and clone the catalog project:
sudo git clone https://github.com/boisalai/[your-project-name-on-github].git catalog.
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.
- In `__init__.py`, replace:
```
app.run(host='0.0.0.0', port=5000)
```
By:
```
app.run()
```
- in `database.py`, replace line 58:
```
# engine = create_engine("sqlite:///itemcatalog.db")
engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
```
- Run the files:
```
python database_setup.py
python seeder.py
```

### Authenticate login through Google 
- Go to Google Cloud Plateform.
- Click APIs & services on left menu.
- Click Credentials.
- Create an OAuth Client ID (under the Credentials tab), and add `http://your-public-ip` and `http://your-public-ip.xip.io` as authorized JavaScript origins.
- Add `http://your-public-ip.xip.io/gconnect` and `http://your-public-ip.xip.io/login` as authorized redirect URI.
- Download the corresponding JSON file, open it et copy the contents.
- Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.
- Replace the client ID to line 25 of the `templates/login.html` file in the project directory.
- Update path of client_secrets.json file in `application.py` to:
```
/var/www/catalog/catalog/client_secret.json
```
### Install the virtual environment and dependencies
- While logged in as grader, install pip: `sudo apt-get install python3-pip`.

- Install the virtual environment: `sudo apt-get install python-virtualenv`

- Change to the `/var/www/catalog/catalog/` directory.

- Create the virtual environment: `sudo virtualenv -p python3 venv3`.

- Change the ownership to grader with: `sudo chown -R grader:grader venv3/`.

- Activate the new environment: `. venv3/bin/activate`.

- Install the following dependencies:
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2
```
- Deactivate the virtual environment: `deactivate`.

### Set up and enable a virtual host
- Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file to use Python 3.
```
#WSGIPythonPath directory|directory-1:directory-2:...
WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.6/site-packages
```
- Create `/etc/apache2/sites-available/catalog.conf` and add the following lines to configure the virtual host:
```
<VirtualHost *:80>
	ServerName [your-public-ip]
	ServerAdmin admin@[your-public-ip]
	WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python3.6/site-packages
	WSGIProcessGroup catalog
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
```
- Enable virtual host: `sudo a2ensite catalog`.
- Reload Apache: `sudo service apache2 reload`.

### Set up the Flask application
- Create `/var/www/catalog/catalog.wsgi` file add the following lines:
```
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "..."
```
- Restart Apache: `sudo service apache2 restart`.

### Disable the default Apache site
- Disable the default Apache site: `sudo a2dissite 000-default.conf`.
- Reload Apache: `sudo service apache2 reload`.

### Launch the Web Application
- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to `http://your-public-ip`

## Notes
- My public ip : `157.56.182.79`
- If you want to login with Google use `http://your-public-ip.xip.io` because google login doesn't work without domain name.

## Resources 
- This GitHub Repositories was helpful to me:
  - https://github.com/boisalai/udacity-linux-server-configurationo
  - https://github.com/mulligan121/Udacity-Linux-Configurationo
  - https://github.com/rrjoson/udacity-linux-server-configuration

