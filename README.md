# Linux Server Configuration
_A baseline installation of a Linux distribution on a virtual machine to host our web application._

# Setting up your own Web Server
*--------------------------------------------------------------------------------------------------*
* Log into AWS and create an instance of ubuntu on the [Amazon Lightsail website](https://lightsail.aws.amazon.com/)
* Select the closest region to your location and the pricing plan you desire.
* Give your instance a name and start it up.

# Server details
Public IP address: `35.154.95.143`

Private IP: `172.26.11.216`

SSH port: `22`

*I also created a keypair (on my instance of AWS lightsail), downloaded the .pem(private key) generated, placed in my ~/.ssh folder and used the following command to connect via our own ssh client(git bash)*
```
ssh -i ~/.ssh/fsnd_udacity_project.pem ubuntu@35.154.95.143
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
 * Additionally we could also use `sudo adduser <username> sudo` to add user to `sudo` group.
 * Change contents of the file to the following:
```
grader ALL=(ALL) NOPASSWD:ALL
```

## Set-up SSH keys for user grader
### Generating Key Pairs:
 * On our local machine, using `ssh-keygen` in the terminal we generate a keygen named `udacity_fsnd_key` with no Passphrase.
 * On our VM use `mkdir /home/grader/.ssh` then generate `touch /home/grader/.ssh/authorized_keys`.
 * Back on our local machine, view the contents of our `.pub` file with `cat ~/.ssh/udacity_fsnd_key.pub` and copy it.
 * On the VM use `sudo nano /home/grader/.ssh/authorized_keys` to edit the file and paste the contents.

*I got the Error: Permission Denied(publickey) and solved it by realising that the nano text editor wrapped long lines into new lines during the copy/pasting process - remove these white spaces to make sure the string contains none.*

### File Permissions
> To ensure other users don't gain access to our account
 * `sudo chmod 700 .ssh`
 * `sudo chmod 644 .ssh/authorized_keys`

Successfully login as the `grader` user using the command:
```
ssh -i ~/.ssh/udacity_fsnd_key grader@35.154.95.143
```

## Packages
# Add Package
  * Install package finger `apt-get install finger`

> Check info about user grader using finger with - `finger grader`

## Update all currently installed packages
 * Show list of packages to be updated - `sudo apt-get update`
 * Upgrade the installed packages - `sudo apt-get upgrade`

# SSH Reboot and Restart
* Use `sudo service ssh restart` to restart ssh after changes.
* Use `sudo reboot` to disconnect and restart VM.


#### *Linux File Permissions*
* Octal Permissions (rw-, r--, r--), (6,4,4)
* Change file Group - `sudo chgrp root .bash_history`
* View File as student `cat .bash_history`
* Change file Owner - `sudo chown root .bash_history`
* Change file Onwer back to student - `sudo chown student .bash_history`

#### *Firewalls*
* List of Ports - https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers
* ubuntu default firewall check - `sudo ufw status`
* Block all connections coming in - `sudo ufw default deny incoming`
* Allow all connections outgoing from our server - `sudo ufw default allow outgoing`
* Check status of our ubuntu default firewall again - `sudo ufw status`
* Allow all SSH connections - `sudo ufw allow ssh`
* Allow our Vagrant SSH set up all tcp on port 2222- `sudo ufw allow 2222/tcp`
* Allow support for our basic HTTP server - `sudo ufw allow www`
* Enable our Firewall (*only once we have configured our firewall correctly*) - `sudo ufw enable`
* Check out status to see implementation of firewall using - `sudo ufw status`

============================================

## 03. Linux - Web Application Servers

* Use the `Apache HTTP Server` to respond to HTTP requests and serve a static webpage
* Configure `Apache` to hand-off specific requests to Python providing the ability to develop dynamic websites
* Setup `PostgreSQL` and write a simple Python application that generates a data-driven website

#### Vagrant Pre-requisites (only for Vagrant)
* Add the following line to your `Vagrantfile` in the project directory
```
config.vm.network "forwarded_port", guest: 80, host: 8080
```

On some Windows machines you may have to add this instead
```
config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
```

* Run `vagrant up` or if you are already running vagrant run `vagrant reload` to refresh.

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