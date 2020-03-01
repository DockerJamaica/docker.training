.. Docker Training documentation Angular + MongoDb, created by
   sphinx-quickstart on Fri Feb 28, 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.


Dockerizing a Angular, Node.js, Express and MongoDB Application
====================================================================

In this training, we will focus on dockerizing a node-based Angular + Express
+ application that is using MongoDB as it's database backend.

The application is a Real Estate Listing website. The codebase can be found
at `https://github.com/sjfortin/real-estate-listings.git <https://github.com/sjfortin/real-estate-listings.git>`_.

Normally, the requirements for setting up and running this application are:

- Node
- NPM
- MongoDB

This requires manual installation of Node, NPM and MongoDB on each machine,
which may be tedious and troublesome especially if there are some system
requirements or another version of Node or MongoDB are installed.


Why use docker with the Angular + Express + MongoDB app that has so few requirements?
---------------------------------------------------------------------------------------------
* Reduce cost of installing Node, NPM, MongoDB and setting up the application 
  working environment on each machine.
* Isolating dependencies and source code:
* Remove the need of having multiple version of Node installed on the host
  machine.
* Remove version conflicts of Node packages
* Makes cloud migration easy. Docker runs on all the major cloud providers and
  operating systems, so apps containerized with Docker are portable across
  data centres and clouds.
* It works on everyone’s machine. Docker eliminates the “but it worked on
  my laptop” problem.


Steps to Dockerizing the Real Estate Application:
------------------------------------------------------

Clone the codebase
+++++++++++++++++++++
::

   git clone https://github.com/sjfortin/real-estate-listings.git


Create a Dockerfile
++++++++++++++++++++
In the root folder of the project, create a file 
named ``Dockerfile``. Next, add the following code to your docker file.
**The ``Dockerfile`` will be used to generate the Docker image for 
the application.**

::

    # The first instruction is what image we want to base our container on
    # We Use an official Node version 10 image as a parent image
    FROM node:8.10
    
    # Create an environment variable for MongoDB URI
    ENV MONGODB_URI='mongodb://localhost:27017/realestate'
    
    # Set working directory for the project to /usr/src/app
    # NOTE: all the directives that follow in the Dockerfile will be executed
    # in working directory.
    WORKDIR /usr/src/app
    
    # Copy the contents of the project folder into the working directory of
    # docker image
    COPY . /usr/src/app/
    
    # Install any needed packages specified in package.json
    RUN npm install

    EXPOSE 5000
    
    # Periodically check if the application is running. If not, shutdown the
    # container.
    HEALTHCHECK --interval=2m --timeout=5s --start-period=2m \
      CMD nc -z -w5 127.0.0.1 5080 || exit 1
    
    # Wait 5 seconds for the MongoDB connection
    CMD echo "Warming up" && sleep 5 && npm start

..

    Note: The reason why we choose to use Node version 8.10 image is because some
    of the dependencies for the Angular app are depreciated in newer version
    of node.
    see `https://hub.docker.com/_/node <https://www.google.com/url?q=https://hub.docker.com/_/node>`_



Create a .dockerignore
+++++++++++++++++++++++
In the root folder of the project, create a file 
named ``.dockerignore``. Add the following file and folder names to the
``.dockerignore`` file

::

   .vscode/
   .git/
   dump/
   node_modules/

Any file or folder found in the ``.dockerignore`` file will not be added to the
Docker image when the ``COPY`` directive is executed in the Dockerfile.


Building Angular Docker Image
++++++++++++++++++++++++++++++++++

Run the ``docker build`` command in the current directory 
with ``-t`` to name the Docker image as seen
in the command below::

    docker build -t realestate-angular ./


Setting up the Database and Other Requirements
++++++++++++++++++++++++++++++++++++++++++++++++++

Before we can run the application, we need to setup the MongoDB database
and import the dummy data.
Instead of manually installing MongoDB on our machines, we will be using the
`MongoDB Official Docker Image`_. However,
there are two ways in which we can spin up a new instance of MongoDB. We can
use the ``docker run`` command or ``docker-compose``. In this tutorial, we will
be using the ``docker-compose`` method.

Setting up MongoDB Database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the root folder of the project, create a file 
named ``docker-compose.yml``. In the ``docker-compose.yml`` file, we specify
the version of ``docker-compose`` as ``version 3`` and create a database
service for MongoDB. 

In our database service, we will set a default username, password and database
name for our MongoDB backend. In addition, we will expose the port for our
database service for internal usage.

In the ``server/data`` folder, there are two JavaScript files that are
used to populate the Mongo database. In addition, there two bson files located
in the ``dump/realestate`` folder, which could be used to populate the database
. However, we will not be using any of the sample data, instead, we are going
to setup a new database in mongodb.

Copy the following code to your ``docker-compose.yml`` file.

::

  version: '3' 
  services:
    database:
      image: mongo
      restart: always
      environment:
        MONGO_INITDB_ROOT_USERNAME: root
        MONGO_INITDB_ROOT_PASSWORD: example
        # Create a new database. Please note, the 
        # /docker-entrypoint-initdb.d/init.js has to be executed
        # in order for the database to be created 
        MONGO_INITDB_DATABASE: realestate
      volumes:
        # Add the db-init.js file to the Mongo DB container
        - ./db-init.js:/docker-entrypoint-initdb.d/init.js:ro
      ports:
        - '27017-27019:27017-27019'

..

   Note: The `MongoDB Official Docker Image`_ has a list of environmental
   variables that are used to configure MongoDB.

Next, create the ``db-init.js`` file in the root of the project as seen
in the docker-compose file. Afterwards, add the following code to the file::

  db.createUser({
    user: "user",
    pwd: "secretPassword",
    roles: [ { role: "dbOwner", db: "realestate" } ]
  })
  
  db.users.insert({
    name: "user"
  })

The code above will create a new MongoDB user with the role of database owner.

Now that the database service has been defined, execute the following command
to spin the MongoDB container.

::
   
  docker-compose up -d

..

   Note: The ``docker-compose up`` command creates and runs the container for
   each service that is defined in the ``docker-compose.yml`` file and the
   ``-d`` option runs the container as a daemon (background process)


Afterwards, execute the following command to check if the Mongo DB container is
running.
::

  docker-compose ps


You should see something similar to the following output.
::
              Name                           Command             State                                      Ports                                    
  ----------------------------------------------------------------------------------------------------------------------------------------------------
  real-estate-listings_database_1   docker-entrypoint.sh mongod   Up      0.0.0.0:27017->27017/tcp, 0.0.0.0:27018->27018/tcp, 0.0.0.0:27019->27019/tcp
   

Add Mongo Express Service to Manage MongoDB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that our MongoDB container is running and we can access Mongo databse. We
can add support for Mongo Express. Mongo Express is a lightweight web-based
administrative interface deployed to manage MongoDB databases interactively.

Add the following lines to your ``docker-compose.yml`` file::

    mongo-express:
      image: mongo-express
      restart: always
      ports:
        - 8081:8081
      environment:
        ME_CONFIG_MONGODB_ADMINUSERNAME: root
        ME_CONFIG_MONGODB_ADMINPASSWORD: example
        ME_CONFIG_MONGODB_SERVER: database
      depends_on:
        - database

..

  Note, the ``mongo-express`` service is using the MongoDB user's credentials
  that was set the database service and the database service name.


Running the application in the Docker Container
-------------------------------------------------

At this point, we can run our dockerized application by using the ``docker run``
command, however, for sustanability and simplicity of our software arhitecture
and dependencies, we will be using ``docker-compose`` to run our dockerized 
application.

Before we can run our dockerized application using ``docker-compose``, we need
to create another service in our ``docker-compose.yml`` file to manage our
application. Add the following lines to your ``docker-compose.yml file``::

    web:
      build: .
      image: realestate-angular
      environment:
        # Use the username and password found in the db-init.js file instead
        # of the root username. 
        MONGODB_URI: mongodb://user:secretPassword@database/realestate
      depends_on:
        - database
      ports:
        - 8082:5000


..

   Note: The ``MONGODB_URI`` environmental variable uses the username (root)
   and password (example) in the MongoDB URI that was defined in the database
   service for MongoDB.
   Also, it uses the MongoDB service name (database) as the MongoDB host,
   followed by the database name (realestate).


At this point, your ``docker-compose`` file should look like::

  version: '3' 
  services:
    database:
      image: mongo
      restart: always
      environment:
        MONGO_INITDB_ROOT_USERNAME: root
        MONGO_INITDB_ROOT_PASSWORD: example
        # Create a new database. Please note, the 
        # /docker-entrypoint-initdb.d/init.js has to be executed
        # in order for the database to be created 
        MONGO_INITDB_DATABASE: realestate
      volumes:
        # Add the db-init.js file to the Mongo DB container
        - ./db-init.js:/docker-entrypoint-initdb.d/init.js:ro
      ports:
        - '27017-27019:27017-27019'
  
    mongo-express:
      image: mongo-express
      restart: always
      ports:
        - 8081:8081
      environment:
        ME_CONFIG_MONGODB_ADMINUSERNAME: root
        ME_CONFIG_MONGODB_ADMINPASSWORD: example
        ME_CONFIG_MONGODB_SERVER: database
      depends_on:
        - database
  
    web:
      build: .
      image: realestate-angular
      environment:
        # Use the username and password found in the db-init.js file instead
        # of the root username. 
        MONGODB_URI: mongodb://user:secretPassword@database/realestate
      depends_on:
        - database
      ports:
        - 8082:5000


Execute the following command to run the dockerized application along with
the MongoDB Service::

  docker-compose up -d --build


Afterwards, execute the following command to check if the application and Mongo
DB container are running.
::

  docker-compose ps


You should see something similar to the following output::
   
              Name                           Command             State                                      Ports                                    
  ----------------------------------------------------------------------------------------------------------------------------------------------------
  real-estate-listings_database_1   docker-entrypoint.sh mongod   Up      0.0.0.0:27017->27017/tcp, 0.0.0.0:27018->27018/tcp, 0.0.0.0:27019->27019/tcp
  real-estate-listings_web_1        /bin/sh -c npm start          Up      3000/tcp, 0.0.0.0:8080->5000/tcp   


If you wish to see the logs and output for the application and/or MongoDB, run the
following command::

  # See logs for all services
  docker-compose logs -f
  
  # See logs for only the application service
  docker-compose logs -f web
  
  # See logs for only the MongoDB service
  docker-compose logs -f database


Finally
++++++++++

You can visit http://localhost:8080 to see the application in action.

You can find the finish source code for the project on
`DockerJamaica Github page <https://github.com/DockerJamaica/real-estate-listings>`_


For more information on Docker and Docker Compose, please visit the following
links:

- `Docker <https://docs.docker.com/>`_
- `Docker Compose <https://docs.docker.com/compose/>`_

For list of available Docker and Docker Compose commands:

- `Docker Commands <https://docs.docker.com/>`_
- `Docker Compose Commands <https://docs.docker.com/engine/reference/commandline/cli/>`_


If you find a bug in the project source code or documentation,
you can help us by submitting an issue or submit a Pull Request with
the fix to our Github repositories.

- `Real Estate App repository <https://github.com/DockerJamaica/real-estate-listings>`_
- `Docker Training repository <https://github.com/DockerJamaica/docker.training/blob/master/nodejs/angular-mongodb.rst>`_.



.. _MongoDB Official Docker Image: https://hub.docker.com/_/mongo
