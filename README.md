# Linux Server Configuration
_A baseline installation of a Linux distribution on a virtual machine to host our web application._

# Setting up your own Web Server
*--------------------------------------------------------------------------------------------------*
* Log into AWS and create an instance of ubuntu on the [Amazon Lightsail website](https://lightsail.aws.amazon.com/)
* Select the closest region to your location and the pricing plan you desire.
* Give your instance a name and start it up.

# Server details
Public IP address: `35.154.1.22`

Private IP: `172.26.14.76`

SSH port: `2200`

*I also created a keypair (on my instance of AWS lightsail), downloaded the .pem(private key) generated, placed in my ~/.ssh folder and used the following command to connect via our own ssh client(git bash)*
```
ssh -i ~/.ssh/fsnd_udacity_project.pem ubuntu@35.154.1.22
```

# Start Configuration
> We could connect to our Linux Server securely using our browser.

> When you SSH in, you'll be logged as the `ubuntu` user. When you want to execute commands as root, you'll need to use the `sudo` command to do it.

## Add user - grader 
* As ubuntu, add user `grader` with command: `sudo adduser grader` (password: grader)
* *Note:* Use the `su` command to switch user from `ubuntu` to `grader` with `su - grader`

## Allow sudo commands to user grader
* As the `ubuntu` user - 
 * Access the `/etc/sudoers.d` with `sudo ls /etc/sudoers.d`
 * Create the file `grader`, in my case I copied an existing file and renamed it.
 ```
 sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
 ```
 * Use `sudo nano /etc/sudoers.d/grader` and change contents of the file to the following:
```
grader ALL=(ALL) NOPASSWD:ALL
```
 * Additionally we could also use `sudo adduser <username> sudo` to add user to `sudo` group.

## Set-up SSH keys for user grader
### Generating Key Pairs:
 * On our local machine, using `ssh-keygen` in the terminal we generate a keygen named `udacity_fsnd_key` with no Passphrase.
 * On our VM use `mkdir /home/grader/.ssh` then generate `touch /home/grader/.ssh/authorized_keys`.
 * Back on our local machine, view the contents of our `.pub` file with `cat ~/.ssh/udacity_fsnd_key.pub` and copy it.
 * On the VM use `sudo nano /home/grader/.ssh/authorized_keys` to edit the file and paste the contents.

*I got the Error: Permission Denied(publickey) and solved it by realising that the nano text editor wrapped long lines into new lines during the copy/pasting process - remove these white spaces to make sure the string contains none.*

 * Force Key Based Authentication by editing the file `sudo nano /etc/ssh/sshd_config`.

Successfully login as the `grader` user using the command:
```
ssh -i ~/.ssh/udacity_fsnd_key grader@35.154.1.22
```

### File Permissions
> To ensure other users don't gain access to our account
 * `sudo chmod 700 .ssh`
 * `sudo chmod 644 .ssh/authorized_keys`

## Packages
# Add Package
  * Install package finger `sudo apt-get install finger`

> Check info about user grader using finger with - `finger grader`

## Update all currently installed packages
 * Show list of packages to be updated - `sudo apt-get update`
 * Upgrade the installed packages - `sudo apt-get upgrade`


## Change SSH Port
* On the networking tab of our ubuntu instance on the Amazon lightsail page.
  * Add a custom TCP protocol on port 2200.
  * Add a the HTTP (port 80) and NTP (port 123).
* On our VM, change port from 22 to 2200 by editing `sudo nano /etc/ssh/sshd_config`.
* Restart the SSH service
 * Log in via port 2200
 ```
ssh -i ~/.ssh/udacity_fsnd_key grader@35.154.1.22 -p 2200
```

## Uncomplicated Firewall - UFW
### *Configuring our Firewall*
* Check status on our firewall - `sudo ufw status`
* Block all connections coming in - `sudo ufw default deny incoming`
* Allow all connections outgoing from our server - `sudo ufw default allow outgoing`
* Allow all SSH connections - `sudo ufw allow ssh`
* According to the project rubric, allow the following - 
 ```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
 ```
* Enable our Firewall (*only once we have configured our firewall correctly*) - `sudo ufw enable`
* Check out status to see implementation of firewall using - `sudo ufw status`

# Install and configure to serve our python application

### Installing Apache
> Apache is our web server, its job is to process HTTP requests.
  * Install with `sudo apt-get install apache2`
  * Test by opening our public IP on our browser `http://35.154.1.22`, it should display the Default Apache2 Ubuntu **It works** page.

_We can `cd` to the `var/www/html` directory, to find the html file that is currently being served onto the site_ 

### Installing mod_wsgi
> This is a free tool for serving Python applications from Apache.
> We will also be installing a helper package called python-setuptools.
  * Install with `sudo apt-get install python-setuptools libapache2-mod-wsgi`.

* Restart our Apache server for **mod_wsgi** to load: `sudo service apache2 restart`.

### Install git
> git is our version control software.
  * Install with `sudo apt-get install git`

### Setup our flask project
> Move into the www directory , create a folder to store a clone of our catalog repo from github.
  * Move using `cd` to `/var/www` and create the directory `sudo mkdir FlaskApp`.
  * In the `FlaskApp` directory clone our project repo with
    ```
    sudo git clone https://github.com/Christianq010/fsnd_Item-Catalog-linux-server.git FlaskApp
    ```
### Editing our Project
> I had to make changes to my Catalog Item project, which I have explained on the README of that repo as well.
  * https://github.com/Christianq010/fsnd_Item-Catalog-linux-server
> Install Flask, our virtual environment and our dependencies.
> Used a combination of git commands such as push, pull and commit to sync between edits made locally and the repo on our instance of ubuntu.

#### Setting up our project to run on our Ubuntu server
* Create a catalog.wsgi file - `sudo nano flaskapp.wsgi` in the `/var/www/FlaskApp` directory,
with the following contents:
 ```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
 ```
* Rename project.py to __init__.py `mv application.py __init__.py`

#### Installing a virtual environment, flask and other project dependencies
> Setting up a virtual environment will keep the application and its dependencies isolated from the main system. 
> Changes to it will not affect the cloud server's system configurations.

* Use pip to install virtualenv.
  ```
  sudo apt-get install python-pip
  sudo pip install virtualenv
  ```
*  Install - `sudo apt install python-psycopg2`

* `cd` into our `/var/www/FlaskApp/FlaskApp` folder.
* Create an instance of the virtual environment and activate it
```
sudo virtualenv venv
source venv/bin/activate
```

* Install flask and other dependencies
```
sudo pip install Flask
sudo pip install bleach 
sudo pip install httplib2
sudo pip install requests
sudo pip install oauth2client 
sudo pip install sqlalchemy
```
* Leave the virtual env with `deactivate`.


### Configure / Enable a new virtual host
* Create and edit with the following - `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
```
  <VirtualHost *:80>
      ServerName 35.154.1.22
      ServerAdmin admin@35.154.1.22
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
* Restart the apache service with `service apache2 reload` or `service apache2 restart`
* Enable our new virtual host with `sudo a2ensite FlaskApp`

### Install and Configure our PostgreSQL database
* Install - `sudo apt-get install postgresql postgresql-contrib`
* Log in as user - `postgres` (a user created during installation) and create the following tables.
  * Log in with `sudo su - postgres`
  * Run `psql` in terminal
* Create a user named `catalog` with password `123456`
```sql
CREATE USER catalog WITH PASSWORD '123456';
```
* Give this user the ability to create databases
```sql
ALTER USER catalog CREATEDB;
```
* Then create a database managed by that user
```sql
CREATE DATABASE catalog OWNER catalog;
```
* Connect to our newly created database - `\c catalog`
* once connected to the database, lock down the permissions to only let `catalog` create tables:
```sql
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```
* Log out of the `psql` terminal with `\q`, and then use `exit` to logout/ switch back to our `grader` user.

_I followed the Following posts - https://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html, https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps_



## Tips
### ubuntu Password
> Default ubuntu user is created upon instance.
 * To change password - `sudo passwrd <username>`
### Moving folders, renaming and removing files
 * To move folders in linux use `mv source target` and `mv filename1.txt filename2.txt`.
 * To remove `sudo rm -f -r filename`

# SSH Reboot and Restart
* Use `sudo service ssh restart` or `/etc/init.d/ssh restart` to restart ssh after changes.
* Use `sudo reboot` to disconnect and restart VM.

## Viewing Apache2 Error Logs during set-up
* `sudo journalctl | tail`
* Look into `/var/log/` directories, in my case `/var/log/apache2/error.log`


============================================

## 03. Linux - Web Application Servers

* Use the `Apache HTTP Server` to respond to HTTP requests and serve a static webpage
* Configure `Apache` to hand-off specific requests to Python providing the ability to develop dynamic websites
* Setup `PostgreSQL` and write a simple Python application that generates a data-driven website


#### Install Apache
* Install Apache using your package manager with the following command: `sudo apt-get install apache2`
* Confirm Apache is working by visiting `http://localhost:8080` in your browser
* Apache, by default, serves its files from the `/var/www/html` directory. Explore this directory to edit `index.html`

#### Install mod_wsgi
* Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`
* We then need to configure Apache to handle requests using the WSGI module, `cd` to `/etc/apache2/sites-enabled/000-default.conf`
* This file tells Apache how to respond to requests, where to find the files for a particular site and much more [Apache Documentation](https://httpd.apache.org/docs/2.2/configuring.html).
* Add the following line at the end of the `<VirtualHost *:80>` block, right before the closing `</VirtualHost>`
```
<VirtualHost *:80>
    WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost> 
```
* Restart Apache with `sudo apache2ctl restart`

#### Our First WSGI Application
* Quickly test our Apache configuration by writing a very basic WSGI application (http://wsgi.readthedocs.org/en/latest/).
* Despite having the extension `.wsgi`, these are just Python applications.
* Create the `/var/www/html/myapp.wsgi` file using the command `sudo nano /var/www/html/myapp.wsgi`
* Within this file, write the following application:
```
def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
```
*  Reload (http://localhost:8080) to see your application run.

##### Installing PostgreSQL
* Install PostgreSQL to server your data using the command `sudo apt-get install postgresql`.
* Note : Since you are installing your web server and database server on the same machine, you do not need to modify your firewall settings. Your web server will communicate with the database via an internal mechanism that does not cross the boundaries of the firewall. If you were installing your database on a separate machine, you would need to modify the firewall settings on both the web server and the database server to permit these requests.


#### Additional Resources
* https://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line
* https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
* https://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html