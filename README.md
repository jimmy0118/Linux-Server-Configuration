# Linux Server Configuration

This is the final project for Udacity's Full Stack Web Developer Nanodegree. In this project, the Build Item Catalog Web Application will be hosted by a Ubuntu Linux server on an Amazon Lightsail instance.

You can visit http://13.209.156.161/ for the website deployed.

## Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail

1. Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
2. Click `Create instance`.
3. Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
4. Choose a instance plan.
5. Name your instance.
6. Click the `Create` button to create the instance.

## Step 2: SSH into the server

1. Download private key from the `SSH keys` section in the `Account` section on Amazon Lightsail.
2. Move this private key file into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
3. In terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
4. SSH into to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.209.156.161`.

## Step 3: Update and upgrade installed packages

1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages


## Step 4: Change the SSH port from 22 to 2200

1. Run `sudo nano /etc/ssh/sshd_config` to edit the `sshd_config` file.
2. Change the port number from `22` to `2200`.
3. Save and exit using CTRL+X and confirm with Y.
4. Restart SSH: `sudo service ssh restart`.

## Step 5: Configure the Uncomplicated Firewall (UFW)

1. Check firewall status: `sudo ufw status`.
2. Set default firewall to deny all incomings: `sudo ufw default deny incoming`.
3. Set default firewall to allow all outgoings: `sudo ufw default allow outgoing`.
4. Allow incoming TCP packets on port 2200 to allow SSH: `sudo ufw allow 2200/tcp`.
5. Allow incoming TCP packets on port 80 to allow www: `sudo ufw allow www`.
6. Allow incoming UDP packets on port 123 to allow NTP: `sudo ufw allow 123/udp`.
7. Close port 22: `sudo ufw deny 22`.
8. Enable firewall: `sudo ufw enable`.
9. Check out current firewall status: `sudo ufw status`. The output should be like this:
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

10. Update the firewall configuration on Amazon Lightsail website under `Networking`.
Delete default SSH port 22 and add `port 80, 123, 2200`.
11. Open up a new terminal and ssh in via the new port 2200: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.209.156.161 -p 2200`

## Step 6: Create a new user account named `grader`

1. Create a new user: `sudo adduser grader`.
2. Enter a password (twice) and fill out information for this new user.
3. Edits the sudoers file: `sudo visudo`.
4. Search for the line that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

5. Below this line, add a new line to give sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

6. Save and exit using CTRL+X and confirm with Y.
7. Verify that `grader` has sudo permissions. Run `su - grader`, enter the password,
run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for grader on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User grader may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
      (ALL : ALL) ALL
  ```

## Step 7: Create an SSH key pair for `grader` using the `ssh-keygen` tool

1. On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
2. On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
  - Restart SSH: `sudo service ssh restart`
3. On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@13.209.156.161`.

## Step 8: Configure the local timezone to UTC

1. While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. You should see something like that:

  ```
  Current default time zone: 'America/Montreal'
  Local time is now:      Thu Oct 19 21:55:16 EDT 2017.
  Universal Time is now:  Fri Oct 20 01:55:16 UTC 2017.
  ```

## Step 9: Install and configure Apache to serve a Python mod_wsgi application

1. While logged in as `grader`, install Apache: `sudo apt-get install apache2`.
2. Go to http://13.209.156.161/, if Apache is working correctly, a Apache2 Ubuntu Default Page will show up.

## Step 10: Install and configure Python mod_wsgi
1. Install the **mod_wsgi** package: `sudo apt-get install libapache2-mod-wsgi python-dev`.
2. Enable **mod_wsgi**: `sudo a2enmod wsgi`.
3. Restart **Apache**: `sudo service apache2 restart`.
4. Check if Python is installed: `python`.

## Step 11: Install and configure PostgreSQL

1. While logged in as `grader`, install PostgreSQL:
 `sudo apt-get install postgresql`.
2. PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

3. Switch to the `postgres` user: `sudo su - postgres`.
4. Open PostgreSQL interactive terminal with `psql`.
5. Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

6. List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

7. Exit psql: `\q`.
8. Switch back to the `grader` user: `exit`.
9. Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
10. Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
11. Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

12. Below this line, add a new line to give sudo privileges to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

13. Save and exit using CTRL+X and confirm with Y.
14. Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for catalog on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

  User catalog may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
      (ALL : ALL) ALL
  ```

15. While logged in as `catalog`, create a database: `createdb catalog`.
16. Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
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
17. Exit psql: `\q`.
18. Switch back to the `grader` user: `exit`.

## Step 12: Install git

1. While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Step 13: Clone and setup the Item Catalog project from the GitHub repository

1. Run `sudo apt-get install git`
2. Create dictionary: `mkdir /var/www/catalog`
3. CD to this directory: `cd /var/www/catalog`
4. Clone the catalog app: `sudo git clone https://github.com/jimmy0118/build_item_catalog.git catalog`
5. Change the ownership: `sudo chown -R grader:grader catalog/`
6. CD to `/var/www/catalog/catalog`
7. Change file `project.py` to `__init__.py`: `mv project.py __init__.py`
8. Change line `engine = create_engine('sqlite:///consolegameswithusers.db',
    connect_args={'check_same_thread':False})` to `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')` in `__init__.py` file
9. Change line `app.run(host='0.0.0.0', port=5000)` to `app.run()` in `__init__.py` file

## Step 14: Authenticate login through Google
1. Go to Google Cloud Plateform.
2. Click APIs & services on left menu.
3. Click Credentials.
4. Create an OAuth Client ID (under the Credentials tab), and add http://13.209.156.161 as authorized JavaScript origins.
5. Download the corresponding JSON file, open it et copy the contents.
6. Open /var/www/catalog/catalog/client_secrets.json and paste the previous contents into the this file.
7. Replace the client ID of the templates/login.html file in the project directory

## Step 15: Deploying a Flask App on Ubuntu VPS
1. Install pip: `sudo apt-get install python-pip`
2. Install packages:
  ```
     sudo pip install httplib2
     sudo pip install requests
     sudo pip install --upgrade oauth2client
     sudo pip install sqlalchemy
     sudo pip install flask
     sudo apt-get install libpq-dev
     sudo pip install psycopg2

   ```

## Step 16: Setup and enble a virtual host
1. Create file: `sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
  ```
     <VirtualHost *:80>
  		ServerName 13.209.156.161
  		ServerAdmin admin@xx.xx.xx.xx
  		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
  		<Directory /var/www/catalog/catalog/>
  			Order allow,deny
  			Allow from all
  			Options -Indexes
  		</Directory>
  		Alias /static /var/www/catalog/catalog/static
  		<Directory /var/www/catalog/catalog/static/>
  			Order allow,deny
  			Allow from all
  			Options -Indexes
  		</Directory>
  		ErrorLog ${APACHE_LOG_DIR}/error.log
  		LogLevel warn
  		CustomLog ${APACHE_LOG_DIR}/access.log combined
     </VirtualHost>
  ```
3. Run `sudo a2ensite catalog` to enable the virtual host
4. Restart `Apache`: `sudo service apache2 reload`

## Step 17: Configure .wsgi file
1. Create file: `sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
  ```
     #!/usr/bin/python
     import sys
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0, "/var/www/catalog/catalog/")
     sys.path.insert(1, "/var/www/catalog/")

     from catalog import app as application
     application.secret_key = 'super_secret_key'
  ```
3. Restart **Apache**: `sudo service apache2 reload`

## Step 18: Edit the database path
1. Replace lines in `database_setup.py`, and `basic_consolewithusers.py` with `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`

## Step 19: Disable defualt Apache page
1. `sudo a2dissite 000-defualt.conf`
2. Restart **Apache**: `sudo service apache2 reload`

## Step 20: Set up database schema
1. Run `sudo python database_setup.py`
2. Run `sudo python lotsofitems.py`
3. Restart **Apache**: `sudo service apache2 reload`
4. Now follow the link to http://13.209.156.161/  the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)

## Sources
1. [Amazon Lightsail Website](https://aws.amazon.com/lightsail/?p=tile)
2. [Google API Concole](https://console.cloud.google.com/)
3. [Udacity](https://www.udacity.com)
4. [Apache](https://httpd.apache.org/docs/2.2/configuring.html)
