Creating a New Django Site on DigitalOcean
==========================================


`Online guide to setup Django, Postgres, Nginx <https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04>`_


Setting Up Postgresql and Django
--------------------------------

First login using SSH as siteadmin

.. code-block:: bash

  $ ssh siteadmin@clappets.com


Install all prerequisite for python3 installation

.. code-block:: bash

  $ sudo apt update
  $ sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl


Login as user *postgres* and issue command *psql* to start a console to query postgresql

.. code-block:: bash

  $ sudo -u postgres psql

Setup Database by name **vanguard** and Database user by the name **django** and password **abracadabra**

.. code-block:: bash

  postgres=# CREATE DATABASE vanguard;
  postgres=# CREATE USER django WITH PASSWORD 'abracadabra';
  postgres=# ALTER ROLE django SET client_encoding TO 'utf8';
  postgres=# ALTER ROLE django SET default_transaction_isolation TO 'read committed';
  postgres=# ALTER ROLE django SET timezone TO 'UTC';
  postgres=# GRANT ALL PRIVILEGES ON DATABASE vanguard TO django;
  postgres=# \q


Upgrade pip and install the package by typing:

.. code-block:: bash

  $ sudo -H pip3 install --upgrade pip
  $ sudo -H pip3 install virtualenv


Make Project Directory and install a virtual environment and install Django, gunicorn, psycopg2


.. code-block:: bash

  $ mkdir ~/vanguard
  $ cd ~/vanguard
  $ virtualenv venv
  $ source venv/bin/activate
  $ pip install django gunicorn psycopg2-binary


Make Django Project

.. code-block:: bash

  $ django-admin.py startproject vanguard ~/vanguard
  $ nano ~/vanguard/vanguard/settings.py

change the following in settings.py file

.. code-block:: bash

  . . .
  ALLOWED_HOSTS = ['clappets.com', 'IP_Address', 'localhost','127.0.0.1']
  . . .


.. code-block:: bash

  . . .
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.postgresql_psycopg2',
          'NAME': 'vanguard',
          'USER': 'django',
          'PASSWORD': 'abracadabra',
          'HOST': 'localhost',
          'PORT': '',
      }
  }
  . . .


.. code-block:: bash

    . . .
    STATIC_URL = '/static/'
    STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
    . . .

complete the set up

.. code-block:: bash

  (venv)$ ~/vanguard/python manage.py makemigrations
  (venv)$ ~/vanguard/python manage.py migrate
  (venv)$ ~/vanguard/python manage.py createsuperuser
  (venv)$ ~/vanguard/python manage.py collectstatic


configure the firewall for port 8000 and run the django development server

.. code-block:: bash

  (venv)$ sudo ufw allow 8000
  (venv)$ ~/vanguard/python manage.py runserver 0.0.0.0:8000


Configure Gunicorn
------------------

Testing Gunicorn

.. code-block:: bash

    (venv)$ cd ~/vanguard
    (venv)$ gunicorn --bind 0.0.0.0:8000 vanguard.wsgi
    (venv)$ deactivate


Creating systemd socket file

.. code-block:: bash

    $ sudo nano /etc/systemd/system/gunicorn_vanguard.socket

Put the following contents in the file, save and close

.. code-block:: bash

    [Unit]
    Description=gunicorn socket for vanguard

    [Socket]
    ListenStream=/run/gunicorn_vanguard.sock

    [Install]
    WantedBy=sockets.target

Creating systemd service file

.. code-block:: bash

    $ sudo nano /etc/systemd/system/gunicorn_vanguard.service

Put the following contents in the file, save and close

.. code-block:: bash

  [Unit]
  Description=gunicorn daemon
  Requires=gunicorn_vanguard.socket
  After=network.target

  [Service]
  User=siteadmin
  Group=www-data
  WorkingDirectory=/home/siteadmin/vanguard
  ExecStart=/home/siteadmin/vanguard/venv/bin/gunicorn \
            --access-logfile - \
            --workers 3 \
            --bind unix:/run/gunicorn_vanguard.sock \
            vanguard.wsgi:application

  [Install]
  WantedBy=multi-user.target


Start and enable the gunicorn socket

.. code-block:: bash

  $ sudo systemctl start gunicorn_vanguard.socket
  $ sudo systemctl enable gunicorn_vanguard.socket


Check status and existence of sock file

.. code-block:: bash

  $ sudo systemctl status gunicorn_vanguard.socket
  $ file /run/gunicorn_vanguard.sock
  $ sudo journalctl -u gunicorn_vanguard.socket


Checking socket activation

.. code-block:: bash

  $ sudo systemctl status gunicorn_vanguard
  $ curl --unix-socket /run/gunicorn_vanguard.sock localhost
  $ sudo systemctl status gunicorn_vanguard
  $ sudo journalctl -u gunicorn_vanguard


Reloading after making changes to service file

.. code-block:: bash

  $ sudo systemctl daemon-reload
  $ sudo systemctl restart gunicorn_vanguard



Configure Nginx
---------------

.. code-block:: bash

  $ sudo nano /etc/nginx/sites-available/vanguard


.. code-block:: bash

  server {
      listen 80;
      server_name clappets.com;
      location = /favicon.ico { access_log off; log_not_found off; }
      location /static/ {
        root /home/siteadmin/vanguard;
      }
      location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn_vanguard.sock;
      }
  }


.. code-block:: bash

  $ sudo ln -s /etc/nginx/sites-available/vanguard /etc/nginx/sites-enabled
  $ sudo nginx -t
  $ sudo systemctl restart nginx
  $ sudo ufw delete allow 8000
  $ sudo ufw allow 'Nginx Full'


Create a new repository and push to Github
------------------------------------------

.. code-block:: bash

  $ echo "# vanguard" >> README.md
  $ git config --global user.email "raheja.sandeep@gmail.com"
  $ git config --global user.name "Sandeep Raheja"
  $ git init
  $ git add *
  $ git commit -m "django initial state"
  $ git remote add origin https://github.com/sandeeprah/vanguard.git
  $ git push -u origin master
