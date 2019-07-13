.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri Jun  7 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Dockerizing Django DRF Real World Application
====================================================
The Django DRF app is an example of a Production-Ready Django JSON API,
which contains real-world examples (CRUD, auth, advanced patterns, etc)
that adheres to the RealWorld API spec.`    


See the project code at:
`https://github.com/DockerJamaica/Docker-Real-World-Examples.git <https://www.google.com/url?q=https://github.com/DockerJamaica/Docker-Real-World-Examples.git&sa=D&ust=1559913830515000>`_


Normally, the requirements for running the Django DRF application are:
* Python 3.7
* Python Virtualenv or PyEnv
* SQLite


Why use docker with the Django DRF app that has so few requirements?
--------------------------------------------------------------------

* Isolating dependencies and source code:
* Remove the need of having multiple version of python installed on the host machine.
* Remove version conflicts of python packages

Steps to Dockerizing the Django DRF application:
------------------------------------------------

Create a Docker file
++++++++++++++++++++
In the root folder of your project, create a file 
named `Dockerfile`. Next, add the following code to your docker file.

::

    # The first instruction is what image we want to base our container on
    # We Use an official Python runtime as a parent image
    FROM python:3.5.7
    
    # The enviroment variable ensures that the python output is set straight
    # to the terminal with out buffering it first
    ENV PYTHONUNBUFFERED 1
    
    # Get the Real World example app
    RUN git clone https://github.com/gothinkster/django-realworld-example-app.git /drf_src
    
    # Set the working directory to /drf
    # NOTE: all the directives that follow in the Dockerfile will be executed in
    # that directory.
    WORKDIR /drf_src
    
    RUN ls .
    
    # Install any needed packages specified in requirements.txt
    RUN pip install -r requirements.txt
    
    VOLUME /drf_src
    
    EXPOSE 8080
    
    CMD python manage.py makemigrations && python manage.py migrate && python manage.py runserver 0.0.0.0:8000
    # CMD ["%%CMD%%"]



The first directive is based on the requirements for running the Django DRF
application, we will need to use a python 3.5 environment. As a result,
we will use a Python 3.5 image. For other Python images,
see `https://hub.docker.com/_/python <https://www.google.com/url?q=https://hub.docker.com/_/python&sa=D&ust=1559913830518000>`_

The next directive `ENV PYTHONUNBUFFERED 1` is an environment variable,
which instructs Docker not to buffer the output from Python in the standard
output buffer, but simply send it straight to the terminal.


Afterwards, the RUN command executes the git clone command on Real-World
Django repository and set the folder name to django_drf. The django_drf folder
is then set as the working directory and all the directives that follow in
the Dockerfile will be executed in that directory.

Lastly, the requirements for the application is installed via pip.

Build our custom docker image
+++++++++++++++++++++++++++++


With that, we have our Dockerfile for the container image.
You can now build an image based on your application requirements by
running the following command::

    docker build -t django_drf .


If you run the following command you will see the Python:3.5.7 image
\along with your image, `django_drf`::

    docker images


Running your dockerized application
-----------------------------------
Execute the following command to create a container for the django_drf image,
which your application will run in::

    docker run -d -p 8080:8000  -v src:/drf_src --name django_drf_app django_drf


If `docker run` is successful, open your browser and access your API or
application. You should be able to access it at `127.0.0.1:8080/admin`.
If you can see the login page, then your container is up and running.

However, there aren\’t any users for our Django DRF application.
In order to create admin user, we need to access our container to run
the `createuser` command. The following command is used access your container
bash interface::

    docker exec -it django_drf_app /bin/bash


If you got an error stating `Error response from daemon: 
Container XXX is not running`, then run `docker start django_drf_app`,
then run the `docerk exec` command above again.

If your current path changes to `/drf_src`, then you have successfully
access your Django DRF container. Run the `ls -al .` command to list the
files in your project folder. Also, if you run `python --version`, you\’ll 
notice that it is Python 3.5.7.

To add an admin user, you need to run the following code in your container::

    python manage.py createsuperuser



Once you set up your user account, you can log in and check out the application.

Even though Docker makes like easier and solves countless problems,
like any other Tech, it has introduced few issues of its own.
For instance, managing the process of multiple containers.
However, managing Docker containers, processes and resources can easily manage
by the ***Docker Compose*** tool. The `docker-compose` cli can be used to
manage a multi-container for one or more applications. Also, It simplifies
many of Docker options you would enter on the docker run cli.
Docker compose makes it easier for Docker images and workflow to be reused.


Running your dockerized application using docker-compose
--------------------------------------------------------
Unlike our Django DRF example that uses SQLite, a typical API deployment uses
a full RDMS like PostgreSQL or MySQL, it is best to use more than one
containers for production. For instance,\you will need a separate container 
for the web server and a separate container for the database server. 
Docker compose will assist you in specifying how you want the containers to
be built and connected, using a single command. In our case, we can use Docker
compose to tell how our monolithic app should be built.


For us to use Docker Compose, we need to create `docker-compose.yml` file
in the same location as the Dockerfile. Afterwards, add the following lines
to your `docker-compose.yml` file::

    # Specifies which syntax version of Docker compose
    version: '3'
    
    # Build a multiservice arhitecture.
    services:
      # Create a service called web
      web:
        # Build an image from the files in the project root directory (Dockerfile)
        build: .
        # Assigns a name for the container. If no name is specified,
        # Docker will assign the container a random name
        container_name: drf_app
        # Mount the container `/drf` folder to the a `src` folder in the location
        # of the Dockerfile on the host machine.
        volumes:
          - ./src:/drf
        # Map port 8000 to port 9090 so that we can access the application on
        # our host machine by visiting 127.0.0.1:9090
        ports:
          - "9090:8000"


The first line in the `docker-compose.yml` file specifies which syntax version
of Docker compose you want to use.


Next, we define a service called `web` . The `build` directive tells Docker 
compose to build an image from the files in the project root directory. 
The `command` directive is the default command that will be executed when 
Docker runs the container image.


The `container_name` directive assigns a name for the container. 
If no name is specified, Docker will assign the container a random name.
The `volume` directive mounts the `src` folder in the project root directory 
to the container `drf` folder. In essence what this does is; It makes sure that
when I edit any file in the project folder, the container folder is
updated immediately.


Lastly, we expose the port we want to access the container on using 
the ports directive. Please note, the format of the ports directive
is `\<host-port\>:\<container-port\>`.

With that done, you can now build and run the container with the command::

    docker-compose up -d


If your build is successful, open your browser and access your API or
application. You should be able to access it at `127.0.0.1:8080/admin`,
`0.0.0.0:8080/admin` or `localhost:8080/admin`. If you can see the login page,
then your container is up and running.



Please note, there\’s no root URL of the Django DRF application.
Therefore, if you go to 127.0.0.1:8080, you will get a Page Not Found error.
Only the Django admin panel along with API controls are
defined in the application. See the
`url.py <https://www.google.com/url?q=https://github.com/gothinkster/django-realworld-example-app/blob/master/conduit/urls.py&sa=D&ust=1559913830525000>`_ \file to see the list of endpoints or paths that is accessible for the application.



Glossary
+++++++++++

SQLite, a light-weight filesystem RDMS, which doesn\’t have any system
requirements and slower than featured rich RDMS like PostgreSQL and MySQL.


Sources:

* `Building a Production Ready Django JSON API by Derek Howard <https://thinkster.io/tutorials/django-json-api>`


.. toctree::
   :maxdepth: 2
   :caption: Django Training:
   
   known-issues-and-fixes
