# Linux Server Configuration
> Christian Urrea

### IP and URL:
- Server Public IP: 167.99.195.44
- Application URL: http://167.99.195.44.xip.io
- SSH login: ssh -i ./.ssh/udacity -p 2200 grader@167.99.195.44
- Sudo password: grader

### How To:
### Create RSA Key Pair:
Set up a public and private key pair to securely log into the server. The key pair will authenticate you when logging in to the server via SSH. The private key will be kept in your local machine, and the public key will be saved in the server.
1. In a local terminal window, generate the key pair by running  `$ ssh-keygen`
2. Enter file in which to save the key: `/home/user/.ssh/udacity`
3. If you wish to add a level of security - enter a passphrase or if not you may either leave it empty.
You now have a public and private key that for SSH authentication. The key pair is stored inside the *~/.ssh/* directory.
The public key is called *udacity.pub* and the corresponding private key is called *udacity*.


#### Digital Ocean Droplet
1. Log in/create an account on *DigtalOcean*.
2. Create Droplet with image: Ubuntu 18.04 x64
3. Choose a preferred size. I chose the minimum size (1GB/1 vCPU/25GB)
4. Find the section *Add Your SSH Keys*, and paste the content the public key, *udacity.pub* in *SSH content window*. Finalize by adding SSH key.
NOTE: After this step, DigitalOcean will automatically do two thing:
  a. Firstly, create  *~/.ssh/authorized_keys* with appropriate permissions and add your public key to it.
  b. Secondly, it will add the following line to */etc/ssh/sshd_config*: `PasswordAuthentication no`. Essentially disabling password authentication on the root user, and enforcing logins exclusively to SSH.
5. Click `Create Droplet`. After the droplet is created successfully, a public IP address will be assigned. The public IPv4 address of my droplet server is 167.99.195.44

#### Server configuration
1. Log into the droplet server as the root user:
    `$ ssh root@167.99.195.44`
2. Update all packages to their latest versions: `$ apt update && apt upgrade`
3. In case of available kernel update, reboot system: `$ reboot`
4. Change the SSH Port from default 22 to 2200: `$ nano /etc/ssh/sshd_config`
    1. Find line #Port 22. Uncomment the line and and edit to Port 2200. Save file.
    2. Restart SSH server to apply changes: `$ service ssh restart`
5. Configure server time zone to UTC: `$ sudo dpkg-reconfigure tzdata`.
    1. In the menu options, select *None of the Above*, followed by *UTC*

#### Setting Up the Firewall
1.  Configure the firewall to allow only connections for:
    a. SSH port 2200: `$ ufw allow 2200/tcp`
    b. HTTP port 80: `$ ufw allow 80/tcp`
    c. NTP port 123: `$ ufw allow 123/udp`
2. Enable the above firewall rules: `$ ufw enable`

#### Create User grader and give sudo/ssh access
1. While logged in as root: `$ adduser grader`
2. Enter a password (twice) and optionally fill out the new user info form
3. Add grader to group with administrative permissions: `$ usermod -aG sudo grader`
4. Log into grader from root: `su - grader`
5. Enter the following commands to allow SSH access to the user grader:
`$ mkdir .ssh`
`$ chmod 700 .ssh`
`$ cd .ssh/`
`$ touch authorized_keys`
`$ chmod 644 authorized_keys`
6. Now go back to your local terminal and copy the content of the public key file *~/.ssh/udacity.pub*
7. Now in the virtual server terminal, edit *authorized_keys*
`$ sudo nano /.ssh/authorized_keys.` and paste the public key contents to the server's authorized_keys file.

#### Disable Root Login
1. Log into your server as root: `$ ssh -i ~/.ssh/udacity root@167.99.195.44 -p 2200`
2. Open ssh configuration files: `$ nano /etc/ssh/sshd_config`
2. Look for PermitRootLogin line and edit to `PermitRootLogin no`
3. Restart the SSH server `$ sudo service ssh restart`
4. Terminate connection: `$ exit`

#### Install & Conigure Apache Web Server & co
1. login as grader user via SSH: `$ ssh -i ./.ssh/udacity -p 2200 grader@167.99.195.44`
2. Install the Apache Web Server: `$ sudo apt-get install apache2`
3. Confirm whether Apache Web Server was successfully installed, visit http://167.99.195.44 in your Web browser, and you should see the "Apache2 Ubuntu Default Page".
4. Install *mod_wsgi* to allow Python applications to run from Apache server:`$ sudo apt install libapache2-mod-wsgi`
5. Enable *mod_wsgi* module: `$ sudo a2enmod wsgi`
6. Restart Apache Server: `$ sudo service apache2 restart`

#### Install & Configure PostgreSQL
1. Install PostgreSQL: `$ sudo apt-get install postgresql`
2. Switch to the postgres user: `$ sudo su - postgres`
3. Open the psql shell: `$ psql`
4. Run the following commands:
`postgres=# CREATE DATABASE catalog;`
`postgres=# CREATE USER catalog;`
`postgres=# ALTER ROLE catalog WITH PASSWORD 'password';`
`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
5. Exit from the terminal: \q then `exit`

#### Clone Catalog Application
1. Logged in as grader, install git: sudo apt-get install git.
2. Move to `$ cd /var/www/` and make catalog directory `$ mkdir catalog`
3. Set ownership of catalog directory to user grader:  `$ sudo chown -R grader:grader catalog/`
4. Move to catalog directory `$ cd catalog` and clone the catalog project:
`$ sudo git clone https://github.com/chrisaric/item-catalog.git catalog`
5. Move to `$ cd /var/www/catalog/catalog`
6. Rename *application.py* file to *__init__.py*: `$ mv application.py __init__.py.`
7. In *__init__.py*, replace line `app.run(host="0.0.0.0", port=8000, debug=True)` with `app.run()`
8. In *database_setup.py* and *__init__.py* replace the engine line:
`engine = create_engine("sqlite:///catalog.db")` TO
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Set up Google Oauth
1. Go to Google Cloud Platform on Google Developer Console.
2. Click APIs & services on left menu.
3. Click Credentials.
4. Create a new OAuth Client ID (under the Credentials tab),
5. Add http://167.99.195.44 and http://167.99.195.44.xip.io as authorized JavaScript origins.
6. Add http://167.99.195.44.xip.io/oauth2callback
http://167.99.195.44.xip.io/gconnect
http://167.99.195.44.xip.io/gdisconnect
http://167.99.195.44.xip.io/login as authorized redirect URIS

7. Download the client_secrets JSON file and copy its contents
8. Open `/var/www/catalog/catalog/client_secrets.json` and replace the contents with copied one
9. Replace the OAuth client ID in *templates/login.html* with the one for this OAuth.


#### Set Up Pip & Virtual Environment
1. While logged in as grader: install pip package:
    a. for Python 2 run: `$ sudo apt-get install python-pip`
    b. for Python 3 run: `$ sudo apt-get install python3-pip`
2. Install virtual environment with pip: `$ sudo pip install virtualenv`
2. Move to `cd /var/www/catalog/catalog/`
3. Create a virtual environment: `$ sudo virtualenv venv`
4. Set ownership to grader with: `$ sudo chown -R grader:grader venv`
5. Activate the new environment: `$ source venv/bin/activate`
6. Give virtual environment permissions:`$ sudo chmod -R 777 venv`
7. Install application dependencies:
`$ sudo pip install Flask`  
`$ sudo pip install Requests`  
`$ sudo pip install httplib2`  
`$ sudo pip install sqlalchemy`  
`$ sudo pip install psycopg2`  
`$ sudo pip install oauth2client`  
`$ sudo pip install render_template`  
`$ sudo pip install sqlalchemy_utils`  
`$ sudo pip install redirect`  

#### Set up & Enable Virtual Host
1. Create /etc/apache2/sites-available/catalog.conf file
`$ sudo nano /etc/apache2/sites-available/catalog.conf`
and configure the virtual host by pasting below into file:
```
<VirtualHost *:80>
    ServerName 167.99.195.44
    ServerAlias 167.99.195.44.xip.io
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/ven$
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

2. Enable virtual host: `$ sudo a2ensite catalog`

3. Reload Apache server: `$ sudo service apache2 reload`

4. Creatw the *.wsgi* application file to serve as the web server app:
      a. Move to `cd /var/www/catalog/`
      b. Create file catalog.wsgi: `$ sudo nano catalog.wsgi`
      c. Add the following to `catalog.wsgi`

```
activate = '/var/www/catalog/venv/bin/activate_this.py'
with open(activate) as file_:
    exec(file_.read(), dict(__file__=activate))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "super_secret_key"

```

6. Restart Apache server: `$ sudo service apache2 restart`
Now you should be able to run the application at http://206.189.151.124.xip.io/.


#### View App
1. Restart Apache to launch the app$ `sudo service apache2 restart`
2. View app @ http://167.99.195.44.xip.io


### References:
[1] https://www.digitalocean.com/docs/droplets/how-to/

[2] https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04

[3] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

[4] https://github.com/boisalai/udacity-linux-server-configuration

[5] https://github.com/SDey96/Udacity-Linux-Server-Configuration-Project

[6] https://github.com/SruthiV/Linux-Server-Configuration
