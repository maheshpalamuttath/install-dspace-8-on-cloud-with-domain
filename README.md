# Install DSpace 8 on Ubuntu 24.04 LTS

## Table of Contents

1. [Prepare the system for DSpace installation](#prepare-the-system-for-dspace-installation)
2. [Build the Installation Package](#build-the-installation-package)
3. [Install Open JDK](#install-open-jdk)
4. [Install Maven and Ant](#install-maven-and-ant)
5. [Install PostgreSQL](#install-postgresql)
6. [Solr Installation](#solr-installation)
7. [Download the DSpace package](#download-the-dspace-package)
8. [Install Tomcat](#install-tomcat)
9. [Database setup](#database-setup)
10. [Installation of DSpace backend](#installation-of-dspace-backend)
11. [Install the Front End](#install-the-front-end)
12. [Startup the User Interface](#startup-the-user-interface)
13. [Contributors](#contributors)
14. [References](#references)

---

## Prepare the system for DSpace installation

DSpace consists of both a Java-based backend and an Angular-based front end. Both the backend and front end need to be installed separately. Install the prerequisites for the DSpace software. DSpace builds on the Java platform and requires software packages like Tomcat, Solr, Maven, Ant, etc.

### Update the operating system

Apply the following commands one by one to receive new updates. It makes the system new.

```bash
sudo apt update && sudo apt upgrade -y
```

### Create a DSpace user

Create a DSpace user from the terminal after the installation. Here are the commands to create the DSpace user;

```bash
sudo useradd -m dspace
sudo passwd dspace
```

`[enter a password for the new user dspace]`

Add dspace user to sudoers group

```bash
sudo usermod -aG sudo dspace
```

### Create the directory for the DSpace installation

```bash
sudo mkdir /dspace
```

Change the dspace folder permission to the dspace user.

```bash
sudo chown dspace /dspace
```

---

## Build the Installation Package

Install packages to support the DSpace installation.

```bash
sudo apt install wget curl git build-essential mousepad zip unzip -y
```

---

## Install Open JDK

The JDK is a development environment for building applications, applets, and components using the Java programming language. JDK prepares an environment to run Java-based applications on the machine.

```bash
sudo apt install openjdk-17-jdk -y
```

### Set the JAVA_HOME Environment Variable

DSpace requires the Java installation location. Open the following file to add the java environment variable,

```bash
sudo nano /etc/environment
```

Add the following two lines to the file,

```
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
JAVA_OPTS="-Xmx512M -Xms64M -Dfile.encoding=UTF-8"
```

Save and close the file.

Apply the following commands one by one to check Java Home is successfully saved,

```bash
source /etc/environment
echo $JAVA_HOME
echo $JAVA_OPTS
```

---

## Install Maven and Ant

Apache Maven requires for the build process during the DSpace installation. Maven helps to download the dependencies. Maven is used to build the installer. Ant used to install/deploy DSpace to the installation directory

```bash
sudo apt install maven ant -y
```

---

## Install PostgreSQL

PostgreSQL, also known as Postgres, is a free and open-source relational database management system. DSpace makes use of PostgreSQL as a Relational Database System. Apply the following command to install PostgreSQL and related packages;

```bash
sudo apt-get install postgresql postgresql-client postgresql-contrib libpostgresql-jdbc-java -y
```

Apply the following command to check the PostgreSQL version number.

```bash
psql -V psql
```

Check whether the installation of PostgreSQL is successful or not,

```bash
sudo pg_ctlcluster 16 main start
sudo systemctl status postgresql
```

Apply the key, `CTRL + C`, to exit from the screen.

### Create a password for PostgreSQL

```bash
sudo passwd postgres
```

Try to login into PostgreSQL using the password created.

```bash
su postgres
```

The command prompt looks like this `postgres@user:` if you could log in successfully. Apply the following command to exit from there,

```bash
exit
```

Open the following file,

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Uncomment the line (remove `#`) `listen_addresses = 'localhost'` under the connection settings option.
Save and exit

We need to tighten the security of PostgreSQL. Open the following file,

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Find the line, `# Database administrative login by Unix domain socket`.

Add the following lines below it,

```
#DSpace configuration
host dspace dspace 127.0.0.1 255.255.255.255 md5
```

Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

---

## Solr Installation

Solr is an open-source search platform, written in Java. Its major features include full-text search, hit highlighting, faceted search, real-time indexing, dynamic clustering, database integration, NoSQL features, and rich document handling.

Download the Solr software package from the website and save it to the opt folder in the operating system.

Enter into the home folder. Apply the following command,

```bash
cd
```

Download the Solr (version 8) package using the following command line,

```bash
sudo wget https://downloads.apache.org/lucene/solr/8.11.4/solr-8.11.4.zip
```

Unzip the package,

```bash
sudo unzip solr-8.11.4.zip
```

Apply the following command to install Solr,

```bash
sudo bash solr-8.11.4/bin/install_solr_service.sh solr-8.11.4.zip
```

After successful installation, exit from the process. Apply the key combination,

`Ctl + C`

Apply the following commands one by one to start automatically upon system boot.

```bash
sudo systemctl enable solr
sudo systemctl start solr
sudo systemctl status solr
```

Apply `CTRL + C` key combination to exit from Solr status in Terminal.

Add the URL on a browser to open the Solr interface,

```
http://localhost:8983/solr
```

---

## Download the DSpace package

Create a temporary folder with the name build in the root folder (/) to store the DSpace package,

```bash
sudo mkdir /build
```

Download the DSpace package. Enter into the /build directory.

```bash
cd /build
```

Download the DSpace 8 package into /build folder.

```bash
sudo wget https://github.com/DSpace/DSpace/archive/refs/tags/dspace-8.0.zip
```

Extract the package,

```bash
sudo unzip dspace-8.0.zip
```

Change the permission of the /build folder.

```bash
sudo chmod 777 -R /build
```

---

## Install Tomcat

Tomcat provides a "pure Java" HTTP web server environment where Java code can run.

```bash
sudo apt install tomcat10 -y
```

Specify the DSpace installation path to Tomcat. Open the following file,

```bash
sudo nano /lib/systemd/system/tomcat10.service
```

Find the category, `#Security`,

Add the following line at the last portion,

```
ReadWritePaths=/dspace
```

Save and close the file.

Alter Tomcat's default configuration to support searching and browsing of multi-byte UTF-8.

Open the following file,

```bash
sudo nano /etc/tomcat10/server.xml
```

Find the below-mentioned lines in the file, and comment out. Add `<!--` in the first line and `-->` in the last line,

```xml
<Connector port="8080" protocol="HTTP/1.1"
           	connectionTimeout="20000"
           	redirectPort="8443" />
```

It will look like this after commented out,

```xml
<!-- <Connector port="8080" protocol="HTTP/1.1"
           	connectionTimeout="20000"
           	redirectPort="8443" /> -->
```

Add the following lines below the commented-out lines,

```xml
<Connector port="8080" protocol="HTTP/1.1"
			minSpareThreads="25"
			enableLookups="false"
			redirectPort="8443"
			connectionTimeout="20000"
			disableUploadTimeout="true"
			URIEncoding="UTF-8"/>
```

Save and close the file.

Restart Tomcat,

```bash
sudo systemctl restart tomcat10.service
```

If it shows the following message,

```
Warning: The unit file, source configuration file or drop-ins of tomcat9.service changed on disk. Run 'systemctl daemon-reload to reload units.
```

Apply the following command,

```bash
sudo systemctl daemon-reload
```

Again try to restart the Tomcat,

```bash
sudo systemctl restart tomcat10.service
```

---

## Database setup

Create a DSpace user. Apply the following commands one by one,

Login into the Postgres terminal,

```bash
sudo su postgres
```

Enter the following directory.

```bash
cd /etc/postgresql/16/main
```

Create the user named dspace. Enter the password when it asks. This is the connection password for the DSpace database; note it down.

```bash
createuser --username=postgres --no-superuser --pwprompt dspace
```

This command will create the database with the name dspace. The database owner is dspace, the user created in the previous command.

```bash
createdb --username=postgres --owner=dspace --encoding=UNICODE dspace
```

### Enable pgcrypto extension

The pgcrypto module provides cryptographic functions for PostgreSQL. It secures data from the security breach.

```bash
psql --username=postgres dspace -c "CREATE EXTENSION pgcrypto;"
```

Exit from the Postgres terminal,

```bash
exit
```

### Create a new DSpace configuration file

Enter into the folder,

```bash
cd /build/DSpace-dspace-8.0/dspace/config
```

Make a copy of the local.cfg file,

```bash
sudo cp local.cfg.EXAMPLE local.cfg
```

Open the local.cfg file,

```bash
sudo nano local.cfg
```

See the essential lines in the file and their purpose. Give IP address or domain name instead of localhost.

```properties
# DSpace server URL
dspace.server.url = http://localhost:8080/server

# DSpace frontend URL
dspace.ui.url = http://localhost:4000

# Name of the site
dspace.name = DSpace at My University
```

If any changes in the DSpace database username/password, mention them here.

```properties
# Database username and password
db.username = dspace
db.password = dspace
```

Uncomment the following line in the local.cfg file,

```properties
solr.server = http://localhost:8983/solr
```

Save and close the file.

---

## Installation of DSpace backend

Login as the Root user,

```bash
sudo su
```

Enter into the DSpace package downloaded in /build folder.

```bash
cd /build/DSpace-dspace-8.0
```

Download the dependencies and the required packages to build DSpace,

```bash
mvn package
```

The package-building process depends on the Internet speed and server responses and it may take 10 to 20 minutes to finish.

### Install DSpace Backend

Enter into the folder where DSpace installer located and install,

```bash
cd dspace/target/dspace-installer
```

Install the DSpace package,

```bash
ant fresh_install
```

After successful install, exit from sudo user, apply the following command,

```bash
exit
```

Copy the DSpace web apps folder to the Tomcat server,

```bash
sudo cp -R /dspace/webapps/* /var/lib/tomcat10/webapps
```

Copy DSpace Solr folders into the default home of Solr (/var/solr/data),

```bash
sudo cp -R /dspace/solr/* /var/solr/data
```

Change the permission of the folder.

```bash
sudo chown -R solr:solr /var/solr/data
```

Restart Solr

```bash
sudo systemctl restart solr
```

Open the Solr on the browser using the URL, `http://localhost:8983` and check the Core Selector is available on the Left side of the screen.

### Initialize the Database

Enter into the below mentioned folder and apply the migration command.

```bash
cd /dspace/bin/
sudo ./dspace database migrate
```

### Create a DSpace Administrator Account

```bash
sudo /dspace/bin/dspace create-administrator
```

This process will ask questions,

```
Creating an initial administrator account
E-mail address: e.g. dspace@localhost
First name: e.g. DSpace
Last name: e.g. MGU
Is the above data correct? (y or n): y
Password will not display on screen.
Password: Enter the password to login DSpace.
Again to confirm: Confirm the password again.
Administrator account created
```

Change permission of DSpace to Tomcat user

```bash
sudo chown -R tomcat:tomcat /dspace/
sudo systemctl restart tomcat10.service
```

Now Open a Browser and test whether the below pages are displaying correctly or not.

Open the REST API Interface at,

```
http://localhost:8080/server
```

Try to open OAI-PMH Interface,

```
http://localhost:8080/server/oai/request?verb=Identify
```

---

## Install the Front End

To install the front required to install Node.js, Yarn, and DSpace-Angular packages.

### Install Node.js

Node.js is an open-source, cross-platform, back-end JavaScript runtime environment.

```bash
sudo apt install nodejs -y
```

### Install npm

```bash
sudo apt install npm
```

### Install Yarn

Yarn is a software packaging system.

```bash
sudo npm install --global yarn
sudo npm install --global pm2
```

### Front end installation

DSpace user interfaces build on Angular. Angular is a TypeScript-based free and open-source web application framework. Follow the steps to install DSpace-Angular.

We are going to place the dspace-angular package into dspace home folder. Enter it into the folder;

```bash
sudo su
cd /home/dspace
```

Download the dspace-angular package,

```bash
sudo wget https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-8.0.zip
```

Extract the package,

```bash
unzip dspace-8.0.zip
```

Remove the dspace zip package,

```bash
rm dspace-8.0.zip
```

Enter into the dspace-angular folder,

```bash
cd /home/dspace/dspace-angular-dspace-8.0
```

Install all dependencies,

```bash
yarn install
```

Enter into the /dspace-angular/config folder,

```bash
cd config
```

Copy and rename the following file,

```bash
cp config.example.yml config.prod.yml
```

Open the config file

```bash
nano config.prod.yml
```

Find the block of information related REST API server settings.

Make changes like this,

```yaml
# The REST API server settings
# NOTE: these settings define which (publicly available) REST API to use. They are usually
# 'synced' with the 'dspace.server.url' setting in your backend's local.cfg.
rest:
  ssl: false
  host: localhost
  port: 8080
```

Save and close the file.

Run the command,

```bash
yarn run build:prod
exit
```

> If this operation fails, add more RAM to your machine and try again. I recommend a capacity of at least 5-6 GB of RAM.

---

## Startup the User Interface

### Create a PM2 JSON configuration file

Create the file,

```bash
sudo nano /home/dspace/dspace-angular-dspace-8.0/dspace-ui.json
```

Copy the following content into the dspace-ui.json file,

```json
{
    "apps": [
        {
           "name": "dspace-ui",
           "cwd": "/home/dspace/dspace-angular-dspace-8.0/",
           "script": "dist/server/main.js",
           "instances": "max",
           "exec_mode": "cluster",
           "env": {
              "NODE_ENV": "production"
           }
        }
    ]
}
```

Start the application using PM2. Apply the following command to start the service,

```bash
sudo su
pm2 start /home/dspace/dspace-angular-dspace-8.0/dspace-ui.json
```

Create a cronjob entry at the ROOT user to autostart dspace-ui.json. Login as a ROOT user. Apply the following commands,

```bash
crontab -e
```

Add the following line

```
#Auto start dspace-angular
@reboot bash -ci 'pm2 start /home/dspace/dspace-angular-dspace-8.0/dspace-ui.json'
```

To save the file, apply `CTRL + O` key combination. Exit from the editor and apply `CTRL + X`.

Enter the below URL on the browser to start the DSpace front end,

```
http://localhost:4000
```

You may also want to install/configure pm2-logrotate to ensure that PM2's log folder doesn't fill up over time.

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 1000K
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:rotateInterval 0 0 19 1 1 7
```

Build directory no longer required after the installation of DSpace; remove it. Apply the following command;

```bash
sudo rm -rf /build
```

Restart the computer and check the Solr, Tomcat, and DSpace works.


## Apache2 Reverse Proxy with SSL

### 3. Install Apache2

```bash
sudo apt update
sudo apt install apache2 -y
```

Enable the required Apache modules:

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod ssl
sudo a2enmod rewrite
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

### 4. Configure HTTP VirtualHost

Create the Apache configuration file:

```bash
sudo nano /etc/apache2/sites-available/dspace.opensio.co.in.conf
```

Add:

```apache
<VirtualHost *:80>

    ServerName dspace.opensio.co.in

    RewriteEngine On
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

    ErrorLog ${APACHE_LOG_DIR}/dspace_error.log
    CustomLog ${APACHE_LOG_DIR}/dspace_access.log combined

</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite dspace.opensio.co.in.conf
```

### 5. Configure SSL Certificate

Install Certbot:

```bash
sudo apt install certbot python3-certbot-apache -y
```

Request an SSL certificate:

```bash
sudo certbot --apache -d dspace.opensio.co.in
```

Certbot will obtain the certificate and configure an HTTPS VirtualHost.

Verify certificate renewal:

```bash
sudo certbot renew --dry-run
```

### 6. Configure Apache2 Reverse Proxy

Open the SSL VirtualHost created by Certbot:

```bash
sudo nano /etc/apache2/sites-available/dspace.opensio.co.in-le-ssl.conf
```

Inside the existing `<VirtualHost *:443>` block, add the following directives. Preserve the existing SSL certificate and Certbot configuration.

```apache
# Preserve the original hostname
ProxyPreserveHost On

# Forward HTTPS information to the backend
RequestHeader set X-Forwarded-Proto "https"
RequestHeader set X-Forwarded-Host "dspace.opensio.co.in"

# DSpace REST API - Tomcat
ProxyPass /server http://127.0.0.1:8080/server
ProxyPassReverse /server http://127.0.0.1:8080/server

# DSpace Angular UI - PM2
ProxyPass / http://[::1]:4000/
ProxyPassReverse / http://[::1]:4000/
```

> **Important:** Keep the `/server` proxy directives before the `/` directives. Otherwise, REST API requests may be forwarded to the Angular frontend.

The IPv6 loopback address `[::1]` is used because the PM2 frontend listener is bound to IPv6 loopback rather than IPv4 `127.0.0.1`.

### 7. Validate and Reload Apache

Check the configuration:

```bash
sudo apachectl configtest
```

Expected:

```
Syntax OK
```

Reload Apache:

```bash
sudo systemctl reload apache2
```

Check service status:

```bash
sudo systemctl status apache2 --no-pager
```

### 8. Configure DSpace Backend URLs

Open the DSpace backend configuration:

```bash
sudo nano /home/dspace/config/local.cfg
```

Ensure the public URLs are configured as follows:

```properties
dspace.server.url = https://dspace.opensio.co.in/server
dspace.ui.url = https://dspace.opensio.co.in
```

These URLs tell DSpace which public addresses to use for its REST API and frontend.

Restart the backend if you changed its configuration. Use the actual Tomcat service name on your installation:

```bash
sudo systemctl restart tomcat
```

If the service is named differently, identify it with:

```bash
systemctl list-units --type=service | grep -i tomcat
```

### 9. Configure Angular UI

Open the production configuration file:

```bash
cd /home/dspace/dspace-angular-dspace-8.0
nano config/config.prod.yml
```

Check that the UI and REST API settings point to the appropriate addresses:

```yaml
ui:
  ssl: false
  host: localhost
  port: 4000
  nameSpace: /

rest:
  ssl: true
  host: dspace.opensio.co.in
  port: 443
  nameSpace: /server
```

The UI uses HTTP internally because Apache handles public HTTPS. The REST API uses the public HTTPS domain.

> **Note:** If you modify the Angular production configuration, rebuild the UI and restart the PM2 application according to your existing DSpace deployment procedure.

### 10. Verify the Configuration

Check the frontend:

```bash
curl -I https://dspace.opensio.co.in/
```

Expected:

```
HTTP/1.1 200 OK
```

Check the REST API:

```bash
curl -I https://dspace.opensio.co.in/server
```

A redirect to `/server/` is normal.

Check the REST API response:

```bash
curl -s https://dspace.opensio.co.in/server/api | head -c 1500
```

Check the internal Angular UI:

```bash
curl -I 'http://[::1]:4000/'
```

Check the internal REST API:

```bash
curl -I http://127.0.0.1:8080/server
```

---

## Contributors

Express my sincere thanks to the contributors; Matúš Formanek (KMKD FHV University, Slovakia), Jeremiah Kellogg (Library Faculty, Systems, Eastern Oregon University), Anoop P.A. (Technical Assistant, Kerala Institute Of Local Administration), and Mahesh Palamuttath (Librarian, East Campus, Sacred Heart College, Thevara, Ernakulam).

---

## References

- https://wiki.lyrasis.org/display/DSDOC7x/Installing+DSpace#InstallingDSpace-Yarn(v1.x)
- https://gist.github.com/otuoma/4b2b23a6358cae99bd437e7a1492ade1
- https://www.youtube.com/watch?v=Ta-BpqBFJsg
- https://hyperlink.co.ke/2020/08/09/install-dspace-7-on-ubuntu-18-04/
- https://www.youtube.com/watch?v=IRB_oE1JU9Q
