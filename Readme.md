# USFND Secure Linux Server Project

This project requires taking a base distro of linux (physically or via Lightsail) and configuring to host a secure Apache web server which renders a full stack based Item Catalog App which deals with maintenance, creation and editions of items under various categories ( https://github.com/shikharsharma23/UFSNDItemCatalogApp )


## Details of the solution 

#### IP Address of server  / Hosted Application
13.232.159.253
#### SSH Port
2200
#### Username
grader
#### Password / Passphrase used
grader123
### SSH key private key for grader to login
private / confidential


## Tasks and Actions Taken Corresponding to those

#### 1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/). 
* Sign up / Log in to Amazon Lightsail .
* Choose Create new instance
* Choose Ubuntu 16.04
* Instance is free for first month and then charged afterwards
* Create an Instance
* Configure network settings to allow tcp request at 2200
* Start the instance

#### 2. Follow the instructions provided to SSH into your server.
* To SSH into this server we can use web browser client provided by Lightsail
* Alternatively we can ssh using git bash and following command 
* ssh -i privateKey-p 2200 ubuntu@13.232.159.253 where privateKey is path to the private key file which can be downloaded from the Lightsail instance management console

#### 3. Update all currently installed packages.
* sudo apt-get update
* sudo apt-get upgrade

#### 4. Change the SSH port from **22** to **2200**. Make sure to configure the Lightsail firewall to allow it.
* go to lightsail and allow tcp connections from 2200
* sudo nano /etc/ssh/sshd_config (sshd_config stores defualt ssh port)
* change port to 2200
* May need to restart ssh service. 

#### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
* sudo ufw status 
* sudo ufw default deny incoming
* sudo ufw default allow outgoing
* sudo ufw allow 2200/tcp 
* sudo ufw allow www
* sudo ufw allow ntp 
* sudo ufw enable
* sudo ufw status

#### 6. Create a new user account named `grader`.
* sudo adduser grader (password : grader123)

#### 7. Give sudo access to this new user
* sudo ls /etc/sudoers.d (sudoers.d has name of sudo and it includes all other files in sudoers.d directory)
* sudo nano /etc/sudoers.d/grader ( We add our user to sudoers.d)
* add grader ALL=(ALL) NOPASSWD:ALL to the file and save it
* sudo ls /etc/sudoers.d

#### 8. Create an SSH key pair for `grader` using the `ssh-keygen` tool.
* On local / personal system and any linux terminal ( git bash etc) , use ssh-keygen tool and along with a passphrase (grader123 in this case).
* You will also be prompted to enter name for your keys (linuxCourse for this case)
* Once done ssh-keygen will generate two files linuxCourse and linuxCourse.pub which are private and public RSA keys respectively
* Login the server using grader account via password or use su - grader to switch account
* from home directory do the following
* mkdir .ssh
* touch .ssh/authorized_keys
* sudo nano authorized_keys ( nano is text editor)
* Copy contents of public key into this file (The directory and file we just made are where all public keys are stored. Private keys corresponding to these can be used to login via ssh )
* Now lets change permissions of this using 
* chmod 700 .ssh
* cd .ssh
* chmod 644 authorized_keys
* We can now login in grader account using 
* ssh -i linuxCourse -p 2200 grader@13.232.159.253 were linuxCourse is private key we generated for grdaer using ssh-keygen

#### 9. Configure the local timezone to UTC.
* sudo dpkg-reconfigure tzdata
* select none of the above in first list and UTC in second

#### 10. Install and configure Apache to serve a Python mod_wsgi application.
* sudo apt-get install apache2 ( used to host web apps)
* sudo apt-get install libapache2-mod-wsgi-py3 (apache can redirect request to our python applications which can be linked to a app.wsgi file which can be linked to apache by making anew virtual host or specifying in the apache conf. Apache by default starts serving index.html (on port 80) form /var/www/html when we install it. We can change this by adding WSgiScriptAlias of our wsgi application (wsgi has python code inside it ) to the apache conf or creating a new virtual host and disabling default conf). More on this in next sections

#### 11. Install and configure PostgreSQL:
* Our app was based on sqlite. We need to modify it to use postgres and also setup postgres on the server
* In next steps setup a db on postgres
* sudo apt-get install postgresql
* sudo su - postgres
* psql
* CREATE DATABASE catalog;
* CREATE USER catalog;
* ALTER ROLE catalog WITH PASSWORD 'password';
* GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
 * \q
* exit

#### 12. Install Git
* sudo apt-get install git-core
* git config --global user.name "shikharsharma23"
* git config --global user.email "shikharcic23@gmail.com"

#### 13. Deploy the Item Catalog project.

* cd /var/www ( here we make our new app)
* sudo mkdir FlaskApp
* cd FlaskApp
* git clone https://github.com/shikharsharma23/UFSNDItemCatalogApp
* sudo mv ./UFSNDItemCatalogApp./FlaskApp
* cd FlaskApp ( now we are in directory of our app)
* sudo mv application.py __init__.py
* Edit databse_setup,populate_db and __init__.py to use 
  engine = create_engine('postgresql://catalog:password@localhost/catalog')* 
* python3 comes with wsgi installation
* install pip
* install pyscopg2, postgressql, flask and oter necessary modules using pip
* sudo python database_setup.py
* sudo python populate_db.py
* create new virtual host sudo nano /etc/apache2/sites-available/FlaskApp.conf
* add following lines to it
```
<VirtualHost *:80>
        ServerName 13.232.159.253
        ServerAdmin shikharcic23@gmail.com
        WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
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
* enable app using sudo a2ensite FlaskApp
* Item catalog App can now be accessed using this server's IP
