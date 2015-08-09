# Linux/Unix Server Configuration
A baseline installation of a Linux (Ubuntu) distribution on a virtual machine. This project sets up and deploys a web app, secures it from attack vectors, installs updates, and configures databases.

This README proffers a step-by-step walkthrough of Udacity's Linux Configuration project as well as resources and tips.


## 1. App Info and Accessibility

*Public IP Address:* 54.186.19.143 

*SSH Port:* 2200

You can log into the virtual server (on Amazon's cloud) using this shell command:
`$ ssh -p 2200 -i ~/.ssh/udacity_key.rsa root@54.186.19.143`

The deployed web app -- a image sharing app -- can be accessed from the following url:
[http://54.186.19.143](http://54.186.19.143)


## 2. Walkthrough

#### Getting started with your development environment[^1]:
-Download [Git Bash](http://www.git-scm.com/downloads) (necessary for Windows users)

-Create a development environment and write down the unique public ip address that was generated for future reference:
[https://www.udacity.com/account#!/development_environment](https://www.udacity.com/account#!/development_environment)

-Download the private key(s)

-Open Git Bash and copy and paste the following into the terminal:
`$ mv <CURRENT_DIRECTORY>/udacity_key.rsa ~/.ssh/`
where <CURRENT_DIRECTORY> is the directory where you downloaded the private key. 

-Enter:
`$ chmod 600 ~/.ssh/udacity_key.rsa`

-Finally, we can lon into our virtual sever by:
`$ ssh -i ~/.ssh/udacity_key.rsa root@<PUBLIC_IP>`
where <PUBLIC_IP> is the ip address that was generated when you created a new development environment.

#### Creating a new user
-Log in (ssh) to your virtual server if you haven't already.

-When you ssh into the cloud, your hostname is displayed (after string the "@" symbol).
You can also find your host name by entering the following command [^2]:
`$ nano /etc/hostname/`
Write this down. Exit this screen [Ctrl+x].

-Enter the following command:
`$ nano /etc/hosts/`
This will bring up a command-line text editor [nano]; we can edit the hosts file here. On the first line, after "localhost," enter the hostname that you wrote down. Press Ctrl+o to save. Exit nano [Ctrl+x].

-Enter the following[^3]:
`$ sudo adduser grader`
Ubuntu will create a new user and ask you for more info about the user. Enter relevant properties about the user or skip them by pressing ENTER.

-Enter:
`$ sudo adduser grader sudo`

#### Update installed packages:
-Log in (ssh) to your virtual server if you haven't already.

-Enter[^4]:
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

-If some of the packages are not updated (e.g. you get a message like, "The following packages have been kept back: linux-image-virtual"), enter[^5]:
`$ sudo apt-get dist-upgrade`

-Restart Ubutnu:
`$ sudo reboot`

#### Enable firewall and configure ports:
-Log in (ssh) to your virtual server if you haven't already.

-Allow ssh access (so that we can continue to remotely configure our server) -- enter[^6]:
`$ sudo ufw allow ssh/tcp`

-We will be denying all connections except the ones we need[^7]. Before we do that, make sure we allow connections to the ssh port we’re connected to as well as port 2200 (which is the ssh port we will be using later on):
```
$ sudo ufw allow 22
$ sudo ufw allow 2200
```
You should have gotten the response of "Rules updated" for each command.

-Let's now enable the firewall:
`$ sudo ufw enable`
You'll get a warning. Type "yes" and continue.

-Restart Ubuntu:
`$ sudo reboot`

-The people at Ubuntu suggest we make a back-up of our sshd_config before we start making changes to it -- so let's do that. Make a read-only back-up copy located in `/etc/ssh` by entering [^8]:
```
$ sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
$ sudo chmod a-w /etc/ssh/sshd_config.factory-defaults
```

-Let's make changes to the config via nano:
`$ sudo nano /etc/ssh/sshd_config`
Change the line that says "Port 22" to "Port 2200". Save and exit nano.

-Restart sshd before changes take place:
`$ sudo service ssh restart`

*NOTE:* From now on, we cannot ssh into our virtual server using our previous command. From now on, we must specify the port [2200] we want to connect to:
`$ ssh -p 2200 -i ~/.ssh/udacity_key.rsa root@<PUBLIC_IP>`
where <PUBLIC_IP> is the public ip address we have been using.

-Let's finish configuring the firewall. Log in (ssh) if you haven't already. 

-We only want the firewall to accept incoming connection to ports 2200, 80, and 123. We can check the allowed connections to our firewall (called "rules") by entering:
`$ sudo ufw status`

-To delete some previous rules, we would enter: 
`$ sudo ufw delete <existing_rule>`
where `<existing_rule>` is the rule to be deleted. I.e. we prefix our previous rule with the word "delete". 

-Go ahead and delete port 22: 
`$ sudo delete ufw allow 22`
Check the firewall status to see confirm deletions. Go ahead and delete all other ports that are not 2200.

-Finally, add port 80 (HTML) and port 123 (NTP):
```
$ sudo ufw allow 80
$ sudo ufw allow 123
```

#### Configure time zone to UTC [^9]:
-Log in (ssh) to your virtual server if you haven't already.

-Enter the following:
`$ sudo dpkg-reconfigure tzdata`

-You will be presented with a pink screen. Select "None of the above" then select "UTC" in the following screen. Finally, you are prompted with the current local time and the current universal time -- they should be the same.

#### Install and configure Apache to serve a Python mod_wsgi application:
-Log in (ssh) to your virtual server if you haven't already.

-Install Apache; enter[^10]:
`$ sudo apt-get install apache2`

-We need to install several dependencies. First, install "mod_wsgi":
`$ sudo apt-get install libapache2-mod-wsgi`

-This will install a helper-function called "python-setuptools":
`$ sudo apt-get install python-setuptools`

-Restart the Apache server for the changes to take effect:
`$ sudo service apache2 restart`

-You will likely get this warning: "apache2: Could not determine the server's fully qualified domain name, 
using 127.0.0.1 for ServerName". It's not critical. But we can easily change it. Let's create a new Apache config file called "servername"[^11]:
`$ sudo nano /etc/apache2/conf-available/servername.conf`

-Enter the following -- which will change the server's name to "localhost":
`$ ServerName localhost`
Save and exit nano.

Now enable our config file and restart Apache for it to take effect:
```
$ sudo a2enconf servername
$ sudo service apache2 restart
```

#### Install Git and clone GitHub repository[^12]:
-Log in (ssh) to your virtual server if you haven't already.

-Install Git:
`$ sudo apt-get install git`

-Set your name for commits:
`$ git config --global user.name "<YOUR_NAME>"`
where <YOUR_NAME> is your name.

-Set your email address associated with your GitHub account (for commits):
`$ git config --global user.email "<EMAIL_ADDRESS>"`
where <EMAIL_ADDRESS> is the email account associated with your GitHub account.

-Enter:
`$ cd /var/www`

-Clone your GitHub repo app:
`$ git clone <CLONE_URL>`
where <CLONE_URL> is clone url of your GitHub repo.

-You may need to rename the cloned directory. I will assume your app follows the typical Python "package" structure -- for example, after cloning your repo, your app may be structured like so:
```
var/
└── www/
    └── My-Repo-Name/
            ├── start_server.py
            ├── README.md
            └── my_app/
                ├── __init__.py
                ├── api_endpoints.py
                ├── database_setup.py
                ├── views.py
                ├── uploaded/
                └── static/
```
In this case, we would need to rename "My-Repo-Name" to "my_app":
`$ mv /var/www/<My-Repo-Name> /var/www/<my_app>`
where <My-Repo-Name> is the name of the repo directory (i.e. the outter directory) and <my_app> is the name of the app (i.e. the inner directory).

-Let's make GitHub repo inaccessible[^13].
-Cd to `/var/www/<app_folder>/` where <app_folder> is the name of your app.

-Create a ".htaccess" file:
`$ sudo nano .htaccess`

-Paste the following:
`RedirectMatch 404 /\.git`
Save and exit nano.

#### Setup Flask[^14]:
-Log in (ssh) to your virtual server if you haven't already.

-Install additional packages -- enter:
`$ sudo apt-get install libapache2-mod-wsgi python-dev`

-Enable the downloaded package:
`sudo a2enmod wsgi`
(It should already be enabled.)

-We will create a virtual environment for our app.

-Install pip:
`$ sudo apt-get install python-pip`

-Install "virtualenv":
`$ sudo pip install virtualenv`

-Name your new virtual environment -- enter:
`$ sudo virtualenv <VENV>`
where <VENV> is the name of your virtual environment.

-Enable permissions in the new virtual environment[^15]:
`$ sudo chmod -R 777 <VENV>`
where <VENV> is the name of your virtual environment.

-Activate it:
`$ source <VENV>/bin/activate`

-Install Flask inside:
`$ sudo pip install Flask`

-Deactivate the virtual environment:
`$ deactivate`

-Enter:
`$ sudo nano /etc/apache2/sites-available/<app_name>.conf`
where <app_name> is the name of your app (the directory name in /var/www/).
Note: I once again assume your app is structured as a package.

-Paste the following in nano:
```
<VirtualHost *:80>
        ServerName <PUBLIC_IP>
        ServerAdmin admin@<PUBLIC_IP>
        WSGIScriptAlias / /var/www/<app_name>/<app_name>.wsgi
        <Directory /var/www/<app_name>/<app_name>/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/<app_name>/<app_name>/static
        <Directory /var/www/<app_name>/<app_name>/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
where <PUBLIC_IP> is the public ip you generated when you created the development environment, and where <app_name> is the name of the app (same as the name of the app directory). Note: we are specifying a .wsgi file called <app_name>.wsgi -- this can be called anything, but for simplicity, we will be consistent  and use the name of the app. Save and close nano.

Enable the virtual host:
`$ sudo a2ensite <app_name>`
where <app_name> is the name of the app.

-Create the .wsgi file:

-Enter:
`$ sudo /var/www/<app_name>`
where <app_name> is the name of your app.

-Enter:
`$ sudo nano <app_name>.wsgi`

-Paste the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/<app_name>/")

from <app_name> import app as application
application.secret_key = 'simple_key'
```
where <app_name> is the name of your app. Save and exit nano.

-Restart Apache:
`$ sudo service apache2 restart`

#### Install dependencies[^16]:
-Log in (ssh) to your virtual server if you haven't already.

-Activate your virtual envoirnment:
`$ source <VENV>/bin/activate`
where <VENV> is the name of the virtual environment you created.

-Enter:
```
$ pip install httplib2
$ pip install requests
$ sudo pip install flask-seasurf
$ sudo pip install --upgrade oauth2client
$ sudo pip install sqlalchemy
$ sudo apt-get install python-psycopg2
```

#### Install Postgresql[^17]:
-Log in (ssh) to your virtual server if you haven't already.

-Install Postgres:
```
$ sudo apt-get update
$ sudo apt-get install postgresql postgresql-contrib
```

-By default, PostgreSQL does not allow remote connections to the database.

-Create new super user, "catalog":
```
$ sudo adduser catalog
$ sudo adduser catalog sudo
```
Fill in required password/user properties.

-Connect to psql:
```
$ sudo su - catalog
$ psql
```

-Add user and password to postgres:
`CREATE USER catalog WITH PASSWORD '<USER_PASSWORD>';`
where <USER_PASSWORD> is the password (you choose) for user "catalog".

-Give this user privileges to create relations:
`ALTER USER catalog CREATEDB;`

-Create a new database:
`CREATE DATABASE <DB_NAME> WITH OWNER catalog;`
where <DB_NAME> is the name of the database.

-Connect to database:
`\c <DB_NAME>`
where <DB_NAME> is the name of the database.

-Enter:
`REVOKE ALL ON SCHEMA public FROM public;`

-Enter:
`GRANT ALL ON SCHEMA public TO catalog;`

-Disconnect from database:
`\q`

-Quit psql:
`exit`

-Use nano and open every file in your app that has the snippet `create_engine` -- the expression that binds the ORM. For example, both my `__init__.py` and `database_setup.py` files contain the line, `create_engine('postgresql+psycopg2://vagrant:pass@localhost/imagesharing')`. Change any lines that use `create_engine` to [^18]:
`postgresql://<USER_NAME>:<DB_PASSWORD>@localhost/<DB_NAME>`
where <USER_NAME> is the name of the database user (we had earlier specified this user as "catalog"), <DB_PASSWORD> is his password, and <DB_NAME> is the name of the database you specified. Save and exit nano.

-Move to your app folder (or whatever folder contains the `database_setup.py` file):
`$ cd /var/www`

-Create database:
`$ python database_setup.py`

#### Enable Google OAuth2[^16]:
-Open your browser and enter [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi). Enter your public ip address to receive your host name -- copy/write down this name.

-From your browser, go to [Google's Developer Console](https://console.developers.google.com/project).

-In the left-hand menu, navigate to API's and Auth > Credentials > Edit Settings.

-In the "Authorized  JavaScript origins" add the following:
```
http://<HOST_NAME>
http://<PUBLIC_IP>
```
Under "Authorized redirect URIs" add:
```
http://<HOST_NAME>
```
where <PUBLIC_IP> is the public ip address of your app and <HOST_NAME> is the hose name you obtained when entering your ip address in [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi). Update these new settings and close/minimize your browser.

-Open Git Bash and log in (ssh) to your virtual server.

-Open your app's Apache config file:
`$ sudo nano /etc/apache2/sites-available/<app_name>.conf`
where <app_name> is the name of your app.

-Paste in the following in nano, below the line that statement that contains ServerAdmin:
`ServerAlias <HOST_NAME>`
where  <HOST_NAME> is the host name you received from [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi). Save and exit nano.

-Enable the virtual host:
`$ sudo a2ensite <VIRTUAL_HOST_NAME>`
where <VIRTUAL_HOST_NAME> is the name of the virtual host config file you created earlier.

#### Important changes to app:
-Log in (ssh) to your virtual server if you haven't already.

-Navigate to your app's location:
`cd /var/www/<app_name>/<app_name>`
where <app_name> is the name of your app.

-Search your app for every instance of `client_secrets.json`. Replace this string with the full path of the the location of your "client_secrets.json" file. For example, if you encounter a line such as:
`CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`
or a line like:
`oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')`
you would change the lines to:
`CLIENT_ID = json.loads(open(r'/var/www/<app_dir>/<app_dir>/client_secrets.json', 'r').read())['web']['client_id']`
and
`oauth_flow = flow_from_clientsecrets(r'/var/www/<app_dir>/<app_dir>/client_secrets.json', scope='')`
respectively.
Where <app_name> is the name of the app directory, assuming your "client_secrets.json" is located in `/var/www/<app_name>/<app_name>`.

*NOTE: If your app makes modifications to the file structure -- i.e. it adds/deletes files and folders (e.g. an image sharing app such as the one I have provided) -- then several more changes need to be made:

-Cd to `/var/www/<app_name>/<app_name>` and search all of your app files for any instances where relative paths are used for creating/deleting files and folders (e.g. an UPLOAD_FOLDER or a MEDIA_Folder).

-Restart Apache: `$ sudo service apache2 restart`

-Visit your deployed app on a browser -- which is just your public ip. If you are unable to save delete files, open the terminal (and ssh in) and check the traceback:
`$ sudo tail -30 /var/log/apache2/error.log`

-If your traceback contains a 'permission denied' when trying to write/delete, you will need to make changes to permissions/ownership. Here are some resources that solve this issue -- with the first method being the most successful method:

1) [Make all new files in a directory accessible to a group](http://unix.stackexchange.com/questions/12842/make-all-new-files-in-a-directory-accessible-to-a-group?lq=1)[http://unix.stackexchange.com/questions/12842/make-all-new-files-in-a-directory-accessible-to-a-group?lq=1][^19]

2) [How to set default file permissions for all folders/files in a directory?](http://unix.stackexchange.com/questions/1314/how-to-set-default-file-permissions-for-all-folders-files-in-a-directory)[^20]

3) [http://stackoverflow.com/questions/23870808/oserror-errno-13-permission-denied](http://stackoverflow.com/questions/23870808/oserror-errno-13-permission-denied)[^21]

#### Install Extras:

-Let's install the monitor application, Glances[^16][^22]. Log in (ssh) to your virtual server if you haven't already.

-Enter:
```
$ sudo apt-get install python-pip build-essential python-dev
$ sudo pip install Glances
$ sudo apt-get install lm-sensors
$ sudo pip install PySensors
```

-Let's configure our firewall to prevent brute force attacks (repeated login attempts)[^16].

-Install "Fail2ban":
`$ sudo apt-get install fail2ban`

-Enter:
`$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`

-Open the "Fail2ban" config file:
`$ sudo nano /etc/fail2ban/jail.local`

-In the [Default] section, set the following parameters:
```
bantime = 1800
destemail = <EMAIL>
action = %(action_mwl)s
```
where <EMAIL> is the email that you want to use to alert us of break-in attempts.

-In the [ssh] section, set the following parameter:
`port = 2200`
Save and exit nano.

-Install dependencies (N.b. this takes a few minutes -- don't terminate in the middle of the process!):
`$ sudo apt-get install sendmail iptables-persistent`

-Stop and restart the service:
```
$ sudo service fail2ban stop
$ sudo service fail2ban start
```

## 3. Other helpful Bash commands:

-To copy folder contents recursively to another, we use:
`$ cp -r <folder_to_be_coppied>/* <destination_folder>`
where <folder_to_be_coppied> and <destination_folder> are the names of the target and destination folders, respectively.

-To recursively delete a folder and its contents we use:
`$ rm -rf <directory_name>`
where <directory_name> is the name of the directory.

-You can access manual for every shell command by prepending `$ man` -- for example: `$ man ssh` or `$ man nano`.

-Use the command `$ sudo tail -30 /var/log/apache2/error.log` to see the apache's error log/traceback for the last 30 lines.
 
-Restart Apache (and therefore your app): `$ sudo service apache2 restart`.

## 4. References:

[^1]: https://www.udacity.com/account#!/development_environment
[^2]: http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none
[^3]: http://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line
[^4]: http://askubuntu.com/questions/196768/how-to-install-updates-via-command-line
[^5]: http://ubuntuforums.org/showthread.php?t=1110499
[^6]: https://wiki.ubuntu.com/UncomplicatedFirewall
[^7]: http://guides.webbynode.com/articles/security/ubuntu-ufw.html
[^8]: https://help.ubuntu.com/community/SSH/OpenSSH/InstallingConfiguringTesting
[^9]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
[^10]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
[^11]: http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name/396048#396048
[^12]: https://help.github.com/articles/set-up-git/#platform-linux
[^13]: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
[^14]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
[^15]: http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip
[^16]: https://github.com/stueken/FSND-P5_Linux-Server-Configuration
[^17]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
[^18]: http://killtheyak.com/use-postgresql-with-django-flask/
[^19]: http://unix.stackexchange.com/questions/12842/make-all-new-files-in-a-directory-accessible-to-a-group?lq=1
[^20]: http://unix.stackexchange.com/questions/1314/how-to-set-default-file-permissions-for-all-folders-files-in-a-directory
[^21]: http://stackoverflow.com/questions/23870808/oserror-errno-13-permission-denied
[^22]: http://askubuntu.com/questions/563931/cant-isntall-pysensors-sudo-pip-install-pysensors-generates-valueerror-us