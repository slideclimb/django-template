[![Build Status](https://scrutinizer-ci.com/g/PHPirates/django-template/badges/build.png?b=master)](https://scrutinizer-ci.com/g/PHPirates/django-template/build-status/master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/PHPirates/django-template/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/PHPirates/django-template/?branch=master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/1e52d96b4c3b4bc586827a483287ec3c)](https://www.codacy.com/app/PHPirates/django-template?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=PHPirates/django-template&amp;utm_campaign=Badge_Grade)
[![Codeclimate](https://api.codeclimate.com/v1/badges/7ee6b92b294863348693/maintainability)](https://codeclimate.com/github/PHPirates/django-template/maintainability)

# Django template project

## Table of Contents
* Instructions to run the website on a local computer
* Instructions to run the website on a server

#### Makes use of
* PostgreSQL
* The [TinyMCE](https://www.tinymce.com/) editor
* HTML templates
* CSS
* Directory structure from [docs.djangoproject.com](https://docs.djangoproject.com/en/1.11/intro/tutorial01/).

# Instructions to run the website locally

* Create a project, and download the files to that location.
* Probably you want to use a virtual environment.
* Check that the packages in requirements.txt are installed, you may need to download the `mysqlclient` package from [lfd.uci.edu](http://www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python) selecting the 32 bits version for the right python version (even when you are on 64 bit), copy it to the project location and then run `pip install mysqlclient-1.3.13-cp37-cp37m-win32.whl`.
* If you use PyCharm, you can make a Django Server run configuration (instead of running `runserver` all the time). Add an environment variable with name `DJANGO_SETTINGS_MODULE` and value `mysite.settings.development`, assuming the settings are in `development.py` in a folder `settings` in the folder `mysite`. The development settings are for development on your local computer, production settings are for production on the server.
* Tip: If you try running with `DEBUG=False` on your local computer, Django won't serve your static files for you since this is only meant for in production.
* Possibly you need to select your Python interpreter.
* To use a database locally, install PostgreSQL and create a user and then a database; you can use pgAdmin (instructions for pgAdmin 4 2.0) instead of the command prompt to create a user.
 To create a user, right-click on PostgreSQL and choose create Login/Group Role, give it for example the name of your project.
  Create a database by right-clicking on Databases, give it a name for example myproject_db, and under the 'security' tab grant all privileges to the user you just created.
* Replace name, user and password in `DATABASES` in your settings file.
* Use edit - find - replace in path to replace all references to 'mysite' to our own project name. Also rename the 'mysite' module (take care to be consistent with capitalization).
* (Professional edition of PyCharm only) To run `manage.py` tasks, go to settings - Languages and Frameworks - Django and specify your settings file (in this case development, which includes base). Then you can use Tools - Run manage.py Task (`CTRL` + `ALT` + `R`) to run tasks like `migrate`.
* Each time after you made changes in your models in models.py, run `makemigrations` and `migrate` to apply the changes to the database. Do that now.
* Also as `manage.py` task, run `createsuperuser --username myname` to create a superuser (for example you) for your website backend.
* Use the run configuration you made or run `runserver`.
* Access the backend by navigating to [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin)
* (tip) If you already have a local website running, changing the port number allows you to keep things separate.

# Deploying on an Ubuntu 16.04 server
There are a lot of tutorials around, but I have noticed that instructions get obsolete very quickly, so preferably select the latest one you can find which uses exactly all the tools you want. For me that was the one I had to write myself, as below.

We will use a **postgres** database, **gunicorn** to serve the website, and **nginx** to 'reverse proxy' requests from outside to gunicorn.
It makes life easier if you also use **PyCharm**, and **supervisor** to manage gunicorn.

## Buy the necessary services

* You will want to buy a VPS, which is a (virtual) server on which you can install whatever you want. Make sure not to buy something called 'shared hosting' as it probably means you can only upload static files. At the moment a VPS can be as cheap as five euros a month.
* If you don't have a domain yet, you can probably buy it via de same company as you bought the VPS. If you have one, you can probably transfer the management of it to that company. You could also leave it as you have it, and just point the DNS to the ip address of the VPS.

From now on we assume that the server is already up and running and that you can execute `sudo` commands via SSH, for example using `ssh root@xxx.xxx.xxx.xxx` in bash or using Pycharm.

Just in case, run `sudo apt-get update` and `sudo apt-get upgrade` before anything.

## Setting up users and login

* Log in via SSH with the root user. Make a new user with your name with `adduser eve`. Give her root permissions with `usermod -aG sudo eve`.
* Impersonate her with `sudo su - eve`.
* Set up login with a key pair, if needed on your local computer generate keys. View your public key with `cat ~/.ssh/id_rsa.pub` and copy it.
* To put it on the server, use `mkdir ~/.ssh`, `chmod 700 ~/.ssh`, `nano ~/.ssh/authorized_keys` and `chmod 600 ~/.ssh/authorized_keys`.
* Test that it works by opening a new bash window and check that you can login with eve without needing to enter your password.
* Install a text editor, for example `sudo apt-get install nano`.
* Install a firewall, `sudo apt-get install ufw`.
* Allow SSH (22), Postgres (5432), http (80) and https (443) and other things you can think of: `sudo ufw allow xxx`.
* `sudo ufw enable` and check with `sudo ufw status`.

## Local setup
* Add your server to PyCharm in Settings - Build, Execution, Deployment - Deployment, click on the plus icon, choose SFTP, enter the IP address of your server in SFTP host, specify user name and your key file, for Windows probably in `C:\Users\username\.ssh\id_rsa`. Also, if not already done, specify web server root url as `http://ipadress`.
* Make the server the default one by clicking an icon a few to the right of the 'plus' you used to add the server.
* Go to Settings - Tools - SSH Terminal and select the server as Deployment server.
* You should now be able to ssh into your server with Tools - Start SSH Session (assigning a shortcut to this is a good idea).

* If needed point your (sub)domain to the ip address of your server, probably in the settings of the provider where you registered the domains. This can take a few hours to take effect.

* Install the packages we need with `sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx`.

## Setting up postgres
* Start a postgres session with `sudo -u postgres psql`.
* `CREATE DATABASE mysite_db;`
* `CREATE USER mysite WITH PASSWORD '1234';`
* `ALTER ROLE mysite SET client_encoding TO 'utf8';`
* `ALTER ROLE mysite SET default_transaction_isolation TO 'read committed';`
* Now check if the timezone in `settings/base.py` is correct, if not you can modify it to for example `Europe/Amsterdam`. Then `ALTER ROLE mysite SET timezone TO 'Europe/Amsterdam';`
* `GRANT ALL PRIVILEGES ON DATABASE mysite_db to mysite;`
* `\q` to exit
* Update the production database settings in `mysite/settings/production.py`
* Generate a new secret key to enter in the same file. For example using PyCharm's Python console with
```python
from django.utils.crypto import get_random_string

chars = 'abcdefghijklmnopqrstuvwxyz0123456789!@#$%^*(-_=+)'
print(get_random_string(50, chars))
```
* Make sure to keep this file secret. Also don't forget to check `DEBUG = False` in here.
* While you're there, also add your website domain to `ALLOWED_HOSTS` in the base settings. Be sure to add your server's ip address as well if you want to debug with that!
* Lastly, to enable connection from outside, you need to add to `/etc/postgresql/9.5/main/postgresql.conf` just at the end, `listen_addresses = '*'` and in `/etc/postgresl/9.5/main/pg_hba.conf` you need to add `host all all 0.0.0.0/0 md5`.
* Restart postgres with `sudo /etc/init.d/postgresql restart`.
* If at any time you get the error 
```
psycopg2.OperationalError: could not connect to server: Connection refused
	Is the server running on host "x.x.x.x" and accepting
	TCP/IP connections on port 5432?
```
then the previous steps could be the problem (or your firewall is still blocking 5432, of course, or ... ).


## Setting up a virtual environment
* Back on the server, update pip with `pip3 install --upgrade pip`. 
* Check that you're running the python 3 version of pip with `pip -V`
* Ensure ownership of python to install packages with `sudo chown -R eve:eve /usr/local`.
* Install the package with `pip install virtualenv`.
* I happen to want my virtual environments in `/opt/` so I do `cd /opt`.
* Make sure you create a virtual environment with the latest python you installed! Creating a virtual environment appears to be very much liable to change with python versions, so be sure to check online.
* Install venv with `sudo apt-get install python3-venv`.
* I had python 3.5 installed, checked with `python3.5 -V`. Then you should be able to create a virtual environment with `sudo python3.5 -m venv mysite_env`.
* If that does not work, I once solved that under python 3.6 by doing `sudo python3.6 -m venv --without-pip mysite_env`, `source mysite_env/bin/activate`, `sudo apt-get install curl`, then found out curl doesn't run as sudo so did `sudo bash -c "curl https://bootstrap.pypa.io/get-pip.py | python"` then `deactivate`. 
* Every time you want to do something in your virtual environment, activate it with `source mysite_env/bin/activate`. Do so, now.
* Check with `python -V` for correct python version.
* If you did set the virtual environment up without pip, download it with `wget https://bootstrap.pypa.io/get-pip.py`, install with `python3.6 get-pip.py`. 
* Check with `which pip` and `which python` that everything points inside your virtual environment. If you do need to use for example python3.6 instead of python, remember that or fix the `python` command to avoid mistakes.

## Uploading project and installing dependencies
* `cd mysite_env` and `sudo mkdir mysite`, then correct ownership with `sudo chown -R eve:eve /opt/`.
* In PyCharm, go to the deployment settings of your server as before and edit Root path to the directory you just created, so `/opt/mysite_env/mysite`. Under Mappings, specify `/` as Deployment Path. Under Options you can specify to upload changes automatically or if you hit `CTRL+S`.
* Select your project folder in the PyCharm project view and try to upload your files with `CTRL+S` (if you chose that option) or Tools - Deployment - Upload to ...
* If you succeeded, first install `sudo apt-get install python3.5-dev libmysqlclient-dev` which are needed for the `mysqlclient` package.
* If you didn't remember, check with `which python` (with virtualenv activated) where your python hides, then in PyCharm go to Settings - Project Interpreter and add a new remote one, selecting Deployment Configuration and Move Deployment Server, then select the right path to your python.
* PyCharm should warn you about some dependencies from requirements.txt not being installed, do that. Probably PyCharm will also install helper files which can take a long time.
* Make sure you have the remote python selected as interpreter, (you can also check for package updates there), now you can just like before hit Tools - Run Manage.py Task and run `makemigrations` and `migrate` but now both with production settings: so `makemigrations --settings=mysite.settings.production` and also for `migrate`.
* If needed, create superuser just as with local setup, `createsuperuser --username myname`.
* Run `collectstatic --settings=mysite.settings.production` to gather static files for nginx to serve.

## Setting up gunicorn
* We will setup gunicorn such that nginx will be able to redirect requests to gunicorn which is bound to the Django server.
* Put the `gunicorn_start` script in `/opt/mysite_env/bin/` (after changing all the paths, of course), make sure it has executable permissions: create file with `nano /opt/mysite_env/bin/gunicorn_start` (do not use sudo or FileZilla here) and `sudo chmod u+x /opt/mysite_env/bin/gunicorn_start`.

## Setting up supervisor
* We use supervisor to manage the starting and stopping of gunicorn. If your server would crash or for whatever reason is restarted, this makes sure to automatically start your website too.
* Install with `sudo apt-get install supervisor`.
* Put the file `mysite.conf` in `sudo nano /etc/supervisor/conf.d/mysite.conf`.
* Every time after you change such a supervisor config file, you have to do `sudo supervisorctl reread` and `sudo supervisorctl update`. I gathered things to remember like this [below](#remember).
* You can manually restart with `sudo supervisorctl restart mysite`.

## Setting up nginx
* Install with `sudo apt-get install nginx`
* Edit the content of nginx-config into `sudo nano /etc/nginx/sites-available/mysite`.
* Enable your site by making the symbolic link `sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite`
* Remove the symbolic link to the default config, `rm /etc/nginx/sites-enabled/default`
* Create empty log file `mkdir /opt/mysite_env/mysite/logs/` and `touch /opt/mysite_env/mysite/logs/nginx-access.log`.
* Make the socket directory with `mkdir /opt/mysite_env/mysite/run/`.
* If at any time you get some error that the socket does not exist, create an empty socket file `/opt/mysite_env/mysite/run/gunicorn.sock` in the same way. If at any time you get the error that this is not a socket, remove it. A socket is just a text file, with the great usefulness of enabling nginx to talk to gunicorn in a language that they both understand.
* Make sure the lines in the nginx config which point to the ssl certificates are commented.
* Test the syntax of your nginx config file with `sudo nginx -t` and fix any.

## Setting up HTTPS
Because it's not much work and free, just do it.

* You can get an ssl certificate for free, for example from Let's Encrypt. In that case, just follow their [install guide](https://certbot.eff.org/#ubuntuxenial-nginx). If you need to choose, you are not serving files out of a directory on the server. Also, it's possible that after running certbot it has automatically added lines in the nginx config which point to the ssl certificates (multiple times!). You probably want to replace the old lines you commented out previously with these ones.
* Possibly you need to `sudo fuser -k 80/tcp` to clean things up after setting up https. 
* Then start nginx with `sudo service nginx start`.
* In the future, restart nginx with `sudo service nginx restart`. 
* In case that fails, check the logs at `tail /var/log/long.err.log` or `tail /var/log/long.out.log` to view the error.
* Try to reach your website. If it doesn't work, try setting `DEBUG = True` in settings and then `sudo supervisorctl restart mysite`, reload page.

## <a name="remember">To remember</a>
### Django files
After making changes to Django files, run `sudo supervisorctl restart mysite`.
### Django models
After making changes to Django models, in PyCharm start Tools - Run Manage.py Task  and run `makemigrations --settings=mysite.settings.production` and `migrate --settings=mysite.settings.production` (or from the command line, `python3.6 manage.py makemigrations --settings=...`)
### Static files
After making changes to static files run as manage.py task `collectstatic`. If run from the command line, I think you need to activate the virtual environment first.
### Supervisor config
Every time after you change a supervisor config file in `/etc/supervisor/conf.d/mysite.conf`, you have to do `sudo supervisorctl reread` and `sudo supervisorctl update`.
### nginx config
After changing nginx config files in `/etc/nginx/sites-available/mysite`, test syntax with `sudo nginx -t` and run `sudo service nginx restart`.
### logs
nginx logs are viewed with `tail /var/log/long.err.log` or `tail /var/log/long.out.log`, or `tail /opt/mysite_env/mysite/logs/nginx-error.log` as specified in the nginx config file.