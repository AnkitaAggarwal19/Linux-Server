# Linux-Server

## Project Overview

   A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, 
   to include installing updates, securing it from a number of attack vectors and installing/configuring web and 
   database servers

## Major Keypoints

   1 Deploying a web application to a publicly accessible server.
   
   2 Properly securing application ensures, application remains stable and that user’s data is safe.

###### Link to Item Catalog
[ItemCatalog](http://ec2-13-59-63-151.us-east-2.compute.amazonaws.com/restaurant/)

###### IP Address
   13.59.63.151
###### Port
   2200
   
## Steps Followed

1. Create new development environment instance. 

   - Create new instance.
   - Download the private key.
   
2. Launch Virtual Machine and SSH into the server.

   - Move private key file into the desired folder.

   - Change the file rights of the key (Only Owner Can Read and Write.):
   
           chmod 400 ./grader.pem
   
   - SSH into the instance:
   
           ssh -i ./grader.pem root@PUBLIC_IP_ADDRESS

3. Create New User.

   - Add a new user called grader.

         sudo adduser grader

   - Give sudo access to grader.

         sudo nano /etc/sudoers.d/grader

    Add the following text to the the newly created file:

        grader ALL=(ALL:ALL) ALL

   i. Edit the hosts file :

       sudo nano /etc/hosts

   ii. Add the host to hosts file :

       127.0.1.1 ip-XX-XX-XX-XX

4.  Setup SSH keys for grader.

    - To sort out the virtual login issue.

      * Move to sshd_config file

            sudo nano /etc/ssh/sshd-config

      * In the above file make the following changes

           - set password authentication to "yes"

      * To restart the service run the following command.

            sudo service ssh restart

    - On the local system, Go to the directory where you want to save the Key, and run the following command.

           ssh-keygen -t rsa
           
    - Run the following command to install the generated public key on the server.
    
           ssh-copy-id -i udacity.pub grader@PUBLIC_IP_ADDRESS
           
    - Now you are able to log into the remote VM through ssh with the following command: 
    
           ssh -i udacity.rsa grader@XX.XX.XX.XX

5. Enforce key-based authentication | Change the SSH port from 22 to 2200 | Disable login for root user.

    Run  **sudo nano /etc/ssh/sshd_config** .

    Find the PasswordAuthentication line and edit it to no.

    Find the Port line and edit it to 2200.

    Find the PermitRootLogin line and edit it to no, Then save the file.

    Run **sudo service ssh restart** to restart the service.

6. Change timezone to UTC.

       sudo timedatectl set-timezone UTC

7. Updating all packages on the server

       sudo apt-get update
       sudo apt-get upgrade

8. Configure Firewall

    Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80),     
    and NTP (port 123)

       sudo ufw default deny incoming
       sudo ufw default allow outgoing
       sudo ufw allow 2200/tcp
       sudo ufw allow www
       sudo ufw allow ntp
       sudo ufw enable

9. Configure cron scripts to automatically manage package updates.

    - Install unattended-upgrades if not already installed:

          sudo apt-get install unattended-upgrades

    - To enable it, do:

          sudo dpkg-reconfigure --priority=low unattended-upgrades

10. Install and Configure Apache2 and mod-wsgi and Git

        sudo apt-get install apache2 libapache2-mod-wsgi git
        sudo a2enmod wsgi
        
11. Install and configure PostgreSQL

    - Installing PostgreSQL Python dependencies:

           sudo apt-get install libpq-dev python-dev

    - Installing PostgreSQL:

           sudo apt-get install postgresql postgresql-contrib

    - Check if no remote connections are allowed :

           sudo cat /etc/postgresql/9.3/main/pg_hba.conf

    - Login as postgres User (Default User), and get into PostgreSQL shell:

          sudo su - postgres
          psql

    - Create a new User named catalog: # CREATE USER catalog WITH PASSWORD 'sillypassword';

    - Create a new DB named catalog: # CREATE DATABASE catalog WITH OWNER catalog;

    - Connect to the database catalog : # \c catalog

    - Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;

    - Lock down the permissions only to user catalog: # GRANT ALL ON SCHEMA public TO catalog;

    - Log out from PostgreSQL: # \q. Then return to the grader user: $ exit

    - Inside the Flask application, the database connection is now performed with:

          engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')

12. Install Flask and other dependencies

        sudo apt-get install python-pip
        sudo pip install Flask
        sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils requests
        
13. Clone the Catalog app from Github

    - Make a catalog named directory in /var/www

          sudo mkdir /var/www/catalog

    - Change the owner of the directory catalog

          sudo chown -R grader:grader /var/www/catalog

    - Clone the SongCatalog to the catalog directory:

          git clone https://github.com/AnkitaAggarwal19/Item-Catalog.git /var/www/catalog/

    - Change the branch of repo SongCatalog to production:

          cd catalog && git checkout deployment

    - Make a catalog.wsgi file to serve the application over the mod_wsgi. with content:

          touch catalog.wsgi && nano catalog.wsgi

          import sys
          import logging
          logging.basicConfig(stream=sys.stderr)
          sys.path.insert(0, "/var/www/catalog/")

          from project import app as application

      Inside project.py database connection is now performed with:

           engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')

14. Edit the default Virtual File with following content:

        sudo nano /etc/apache2/sites-available/000-default.conf

        <VirtualHost *:80>
          ServerName XX.XX.XX.XX
          ServerAdmin Ankita
          WSGIScriptAlias / /var/www/catalog/catalog.wsgi
          <Directory /var/www/catalog/>
              Order allow,deny
              Allow from all
          </Directory>
          Alias /static /var/www/catalog/catalog/static
          <Directory /var/www/catalog/static/>
              Order allow,deny
              Allow from all
          </Directory>
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

15. Restart Apache to launch the app

        sudo service apache2 restart

