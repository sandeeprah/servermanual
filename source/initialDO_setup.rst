Digital Ocean Initial Server Setup
==================================

Before we set up the digital ocean server it is recommended that we have our SSH keys ready with us. If they are already created we can use them else we need to create them. Follow this tutorial `how to setup SSH keys <https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804>`_

Create the droplet from the console. Upload the SSH Key and use it while generating the droplet. Thus root access will now be set using SSH key. In such a procedure you will not get any mail from digital ocean about password as there is no password based authentication.

Login using ssh

.. code-block:: bash

  $ ssh root@droplet-ip-address

You will not be prompted to change your password however it is recommended that you set your root password now.

.. code-block:: bash

  $ passwd


An online guide from digital ocean `Initial Server Setup <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04>`_ is available that guides you through the process of initial login and setting up including the creation and access configuration of a new sudo user which is going to be actually used for remote logins.

Using the above guide create a new user with name siteadmin

.. code-block:: bash

  $ adduser siteadmin
  $ usermod -aG sudo siteadmin

Check if firewall allows OpenSSH to connect if not configure it to connect with OpenSSH

.. code-block:: bash

  $ ufw app list
  $ ufw allow OpenSSH
  $ ufw enable
  $ ufw status


Configure regular user **siteadmin** to use SSH keys without password.

.. code-block:: bash

  $ rsync --archive --chown=siteadmin:siteadmin ~/.ssh /home/siteadmin

Do not exit or close the terminal which you are using with root. In a new terminal login with the new account **siteadmin**

.. code-block:: bash

  $ ssh siteadmin@droplet-ip-address


If you are successfully logged in then the mission is complete. 

https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars

https://www.digitalocean.com/docs/networking/dns/quickstart/

At your domain registrar site put nameservers of your domain to ns1.digitalocean.com, ns2.digitalocean.com and ns3.digitalocean.com


.. code-block:: bash

  $ sudo apt update
  $ sudo apt install git


.. code-block:: bash

  $ echo "# vanguard" >> README.md
  $ git init
  $ git add README.md
  $ git commit -m "first commit"
  $ git remote add origin https://github.com/sandeeprah/vanguard.git
  $ git push -u origin master
