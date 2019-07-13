.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri Jun  7 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Docker Shortcuts
=======================

Install Docker
--------------

Installing Docker on Debian based distro
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

   sudo apt-get install docker

Allow non-root users to run docker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   sudo usermod -aG docker $USER

Installing Docker on Windows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Go to
https://hub.docker.com/editions/community/docker-ce-desktop-windows,
download and install it. Once Docker is installed, use Docker
Interactive Window or Powershell to use the following commands.

Login to docker
---------------

To use Docker’s public images, you must have a Docker ID or account, go
to https://hub.docker.com/ and register your account if you don’t have
one as yet. Otherwise, run the following command:

.. code:: bash


   docker login

Checking if there’s any container running
-----------------------------------------

.. code:: bash

   docker ps

List all containers, event if they aren’t running

.. code:: bash

   docker ps -a

Running a container
-------------------

Docker run creates a new container from the docker image and starts the
container docker run tutum/hello-world

Naming a container
~~~~~~~~~~~~~~~~~~

.. code:: bash

   docker run --name web1 tutum/hello-world

Running a docker container on a particular port
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

   # docker run --name web2 -p exposePORTt:localPort tutum/hello-world
   docker run --name web2 -p 8080:80 tutum/hello-world

Running docker containers in the background using daemon
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: bash

   docker run -d --name web3 -p 8080:80 tutum/hello-world
   docker run -d --name web4 -p 8081:80 tutum/hello-world

Get logs for a container
------------------------

.. code:: bash

   docker logs web3

Get all logs for all containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   for dlog in `docker ps -a -q`; do docker logs $dlog; done

Stopping a docker container
---------------------------

.. code:: bash

   docker stop web3

Stopping more than one docker container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   docker stop $(docker ps -a -q)

Starting a docker container
---------------------------

.. code:: bash

   docker start web3

Removing a container
--------------------

.. code:: bash

   # docker rm <container-name>
   docker rm web3

Removing all containers
~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   docker rm $(docker ps -a -q)

Stop and remove all containers

.. code:: bash

   docker rm $(docker ps -a -q) & docker stop $(docker ps -a -q)

List docker images
------------------

.. code:: bash

   docker images

Remove a docker image
---------------------

::

   docker rmi wordpress

Remove all images
~~~~~~~~~~~~~~~~~

::

   docker rmi $(docker images -q)


Remove Everything (Images, Containers, Volumes, Networks, etc)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::
   docker system prune -a
   

It will output::

   WARNING! This will remove:
   	- all stopped containers
   	- all volumes not used by at least one container
   	- all networks not used by at least one container
   	- all images without at least one container associated to them
   Are you sure you want to continue? [y/N] y