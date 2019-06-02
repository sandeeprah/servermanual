Creating a New Flask Site on DigitalOcean
=========================================


`Online guide to setup Flask, Gunicorn, Nginx <https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04>`_



Setting Up Flask
----------------

First login using SSH as siteadmin

.. code-block:: bash

  $ ssh siteadmin@clappets.com


Install all prerequisite for python3 installation

.. code-block:: bash

  $ sudo apt update
  $ sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
  $ sudo apt install python3-venv
  $ mkdir ~/vanguard
  $ cd ~/vanguard
  $ python3.6 -m venv venv
  $ source venv/bin/activate
  (venv)$ pip install wheel
  (venv)$ pip install gunicorn flask
  (venv)$ nano ~/vanguard/vanguard.py



Add the following in vanguard.py file

.. code-block:: bash

  from flask import Flask
  app = Flask(__name__)

  @app.route("/")
  def hello():
      return "<h1 style='color:blue'>Hello There!</h1>"

  if __name__ == "__main__":
      app.run(host='0.0.0.0')


configure the firewall for port 5000 and run flask development server


.. code-block:: bash

  (venv)$ sudo ufw allow 5000
  (venv)$ python vanguard.py


Configure Gunicorn
------------------

.. code-block:: bash

    (venv)$ nano ~/vanguard/wsgi.py


.. code-block:: bash

    from vanguard import app

    if __name__ == "__main__":
        app.run()


.. code-block:: bash

    (venv)$ cd ~/vanguard
    (venv)$ gunicorn --bind 0.0.0.0:5000 wsgi:app
    (venv)$ deactivate
    $sudo nano /etc/systemd/system/vanguard.service

Put the following contents in the file, save and close

.. code-block:: bash


  [Unit]
  Description=Gunicorn instance to serve vanguard
  After=network.target

  [Service]
  User=siteadmin
  Group=www-data
  WorkingDirectory=/home/siteadmin/vanguard
  Environment="PATH=/home/siteadmin/vanguard/venv/bin"
  ExecStart=/home/siteadmin/vanguard/venv/bin/gunicorn --workers 3 --bind unix:vanguard.sock -m 007 wsgi:app

  [Install]
  WantedBy=multi-user.target


Start and enable the gunicorn service

.. code-block:: bash

  $ sudo systemctl start vanguard
  $ sudo systemctl enable vanguard


Configure Nginx
---------------

.. code-block:: bash

  $ sudo nano /etc/nginx/sites-available/vanguard


.. code-block:: bash

  server {
      listen 80;
      server_name clappets.com www.clappets.com;

      location / {
          include proxy_params;
          proxy_pass http://unix:/home/siteadmin/vanguard/vanguard.sock;
      }
  }



.. code-block:: bash

  $ sudo ln -s /etc/nginx/sites-available/vanguard /etc/nginx/sites-enabled
  $ sudo nginx -t
  $ sudo systemctl restart nginx
  $ sudo ufw delete allow 5000
  $ sudo ufw allow 'Nginx Full'


Create a new repository and push to Github
------------------------------------------

.. code-block:: bash

  $ echo "# vanguard" >> README.md
  $ git config --global user.email "raheja.sandeep@gmail.com"
  $ git config --global user.name "Sandeep Raheja"
  $ git init
  $ git add *
  $ git commit -m "flask initial state"
  $ git remote add origin https://github.com/sandeeprah/vanguard.git
  $ git push -u origin master

Install wkhtmltopdf (0.12.4) with patched Qt
--------------------------------------------

.. code-block:: bash

  $ cd ~
  $ mkdir
  $ wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz
  $ sudo tar xvf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz
  $ sudo apt install libssl1.0-dev




Note above, that on Ubuntu 18.04 to make https connections work you need to install libssl1.0-dev as given above.
