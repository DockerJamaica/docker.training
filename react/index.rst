.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri July  13 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Dockerizing the React Frontend Real World Applications
=========================================================   

In this training, we will focus on dockerizing the React + Mobx and 
React + Redux frontend codebases.

The React front-end codebases containing real world examples 
(CRUD, auth, advanced patterns, etc) that adheres to the RealWorld spec and API.

Let us start with dockerizing the React + Redux RealWorld example found at:
`https://github.com/gothinkster/react-redux-realworld-example-app`_


Normally, the requirements for setting up and running the React + Redux 
application is:
* Node
* NPM


Why use docker with the React + Redux app that has so few requirements?
--------------------------------------------------------------------------
* Isolating dependencies and source code:
* Remove the need of having multiple version of Node installed on the host machine.
* Remove version conflicts of Node packages
* Makes cloud migration easy. Docker runs on all the major cloud providers and
  operating systems, so apps containterized with Docker are portable across datacenters and clouds.
* It works on everyone’s machine. Docker eliminates the “but it worked on my laptop” problem.


Steps to Dockerizing the React + Redux application:
------------------------------------------------------

Clone the codebase
+++++++++++++++++++++
::

    git clone git@github.com:gothinkster/react-redux-realworld-example-app.git react-redux


Create a Docker file
++++++++++++++++++++
In the root folder of your project, create a file 
named ``Dockerfile``. Next, add the following code to your docker file.

::

    # The first instruction is what image we want to base our container on
    # We Use an official Node version 10 image as a parent image
    FROM node:10
    
    # Create app directory for Real World React example app
    # NOTE: all the directives that follow in the Dockerfile will be executed in
    # that directory.
    WORKDIR /usr/src/app
    
    # Copy the package.json file into our app directory
    COPY . /usr/src/app/
    
    # Install any needed packages specified in package.json
    RUN npm install
    
    RUN ls /usr/src/app
    RUN ls /usr/src/app/public
    
    EXPOSE 3000
    
    CMD npm start

:
    Note: The reason why we choose to use Node version 10 image is because some
    of the dependencies for the React + Redux app are depreciated in newer version
    of node.
    see `https://hub.docker.com/_/node <https://www.google.com/url?q=https://hub.docker.com/_/node>`_


Add a .dockerignore
++++++++++++++++++++++

Add the ``node_modules`` in the .dockerignore file
::

        echo 'node_modules' > .dockerignore


This will speed up the Docker build process as our local dependencies will not be sent to the Docker daemon.


Building the React + Redux Docker Image
+++++++++++++++++++++++++++++++++++++++

Build and tag the Docker image::

    docker build -t react-redux .


If you run the following command you will see the node:10 image 
along with your image, ``react-redux``::

    docker images


Running the React + Redux Docker in a Container
+++++++++++++++++++++++++++++++++++++++++++++++
Execute the following command to create a container from the react-redux image,
which your application will run in::

    docker run -d -p 8092:4100   --name react-redux-app react-redux


If ``docker run`` is successful, open your browser and access your API or
application. You should be able to access it at ``127.0.0.1:8092``.
If you can see the ***conduit homepage***, then your container is up and running.

Setup local backend API via Docker Compose
+++++++++++++++++++++++++++++++++++++++++++++++
By default and for convenience, the application is using the live API server running at 
`https://conduit.productionready.io/api`_ that is provided by the RealWorld project organisers.
You can view the API spec `here <https://github.com/GoThinkster/productionready/blob/master/api>`_ 
which contains all routes & responses for the server.

However, we want to setup a local backend server for our React + Redux app. 
The source code for the backend server (available for Node, Rails and Django)
can be found in the main `RealWorld repo <https://github.com/gothinkster/realworld/>`_,
but we don't have the time to manually setup the backend server. ***This is where Docker shines.***
***We can use the Docker image for the Django backend server that we built in one of our previous trainings
(`https://dockertraining.readthedocs.io/en/latest/django/index.html`_)***

If you've already created the Docker image for the Django DRF backend server, GREAT!!! 

If you haven't, you can use the public Django DRF image on `Docker Hub <https://hub.docker.com/r/realworldio/django-drf>`_

::

    docker pull realworldio/django-drf


Docker's containerization tool helps speed up the development and deployment processes especially when working with microservices.
Docker makes it much easier to link together small, independent services.

We could use ``docker run -d -p 8099:8000 --name django_drf_app realworldio/django-drf`` command to run the Django DRF app in a container 
and update the API url in the React app to match the published localhost port
for the Django DRF app.

However, by using Docker Compose, it will setup the linking between the two containers and ease the pain of manually running various
Docker commands.

Add a docker-compose.yml file
--------------------------------

::

   version: '3' 
   # Build a multiservice arhitecture.
   services:
      # Setup local instance of the Backend Server
      backend:
         # Use the public image for the Django Backend Server
         image: realworldio/django-drf:latest
         # Set the network for the two service so that they can communicate with each other
         networks:
            - reactdrf
         volumes:
            - drf-backend:/drf_src
         # Map port 8000 to port 8199 so that we can access the application on
         # our host machine by visiting 127.0.0.1:8199
         ports:
            - "8199:8000"
      # Create a service called web for the React + Redux app
      web:
         # Build an image from the files in the project root directory (Dockerfile)
         build: .
         depends_on:
            - backend
         # Mount the container `/drf` folder to the a `src` folder in the location
         # of the Dockerfile on the host machine.
         volumes:
            - drf-react-react:/usr/src/app/
         restart: always
         # Map port 3000 to port 8081 so that we can access the application on
         # our host machine by visiting 127.0.0.1:8081
         ports:
            - "8081:3000"
         networks:
            - reactdrf
   networks:
      reactdrf:
   volumes:
      drf-backend:
      drf-react-react:


Update the API URL for the React + Redux app
-------------------------------------------------

In the ``src/agent.js``, change ``API_ROOT`` to the local server's URL (i.e. http://localhost:8199/api)

Build and run the Django + React microservices
------------------------------------------------

::

   docker-compose up -d

Run the following command to verify that the backend and frontend services are running::

   docker ps

You should see something similar to::

    CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                    NAMES
    3764de81cd3d        react-mobx_web                  "docker-entrypoint.s…"   1 minutes ago      Up 1 minutes       0.0.0.0:8081->3000/tcp   react-mobx_web_1
    88a299a68f59        realworldio/django-drf:latest   "/bin/sh -c 'python …"   1 minutes ago      Up 1 minutes       0.0.0.0:8199->8000/tcp   react-mobx_backend_1

If yes, then you should be able to access both site via http://localhost:8199 and http://localhost:8081

***Please note: by default, the Django backend app doesn't have any users. therefore, we need to add a user
to make changes to the frontend React app.***


Adding a backend Admin User
-------------------------------------------
In order to create admin user, we need to access our container to run
the ``createuser`` command. The following command is used access your container
bash interface::

    docker exec -it django_drf_app /bin/bash

To add an admin user, you need to run the following code in your 
container then enter the promoted credentials::

    python manage.py createsuperuser

Afterwards, exit the Docker container running the exit command.
You should be able to access the backend setting by visiting
http://localhost:8199/admin

Codebase
++++++++++

- https://github.com/JamaicanDevelopers/django-realworld-example-app
- https://github.com/JamaicanDevelopers/react-mobx-realworld-example-app
- https://github.com/JamaicanDevelopers/react-redux-realworld-example-app



Sources:
+++++++++

* `Dockerizing a React App by Michael Herman <https://mherman.org/blog/dockerizing-a-react-app/>`
* `Run a React App in a Docker Container by Peter Jausovec <https://mherman.org/blog/dockerizing-a-react-app/>`


.. toctree::
   :maxdepth: 2
   :caption: React Training:

