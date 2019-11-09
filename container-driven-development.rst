.. Docker Training documentation master file, created by
   sphinx-quickstart on Fri July  13 12:02:52 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Container Driven Development
================================

Container driven dedopment is a means by which application is developed, tested
and executed inside a containerized environment from the very beginning of
the application development cycle. 

In this training, we will focus on developing, running and testing a simple
food ordering API built using node inside of a containerized environment.
You'll also understand the HTTP methods (get, post, patch and delete) involved
in creating a standard API.

To follow this article, a basic understanding of Docker, Docker Compose and
JavaScript is required. Please ensure that you have Docker and Docker
Compose installed before you begin. We'll be using NodeJS and the Express
library to build out our application.

Setting Up the App Dependencies
------------------------------------
To get started, we need to create a new folder and create a skeleton 
package.json file. The package.json file is what bootstraps our Node app 
and manages the dependencies of our project.
::
    {
      "name": "order-web-api",
      "version": "1.0.0",
      "description": "Node.js on Docker",
      "author": "Oshane Bailey <b4.oshany@gmail.com>",
      "main": "app.js",
      "scripts": {
        "start": "nodemon app.js"
      },
      "dependencies": {
        "body-parser": "^1.19.0",
        "express": "^4.17.1",
        "nodemon": "^1.19.4"
      }
    }


Next, we'll create a basic NodeJs server configuration. Let's create an app.js
file and add the following code as its content:
::
    const express = require("express");
    
    const app = express();
    const bodyparser = require("body-parser");
    
    const port = process.env.PORT || 3200;
    
    // middleware
    
    app.use(bodyparser.json());
    app.use(bodyparser.urlencoded({ extended: false }));
    
    app.listen(port, () => {
      console.log(`running at port ${port}`);
    });

We're done setting up the app.

Setting Up the Containerized Environment
---------------------------------------------
Create the docker file that uses the node image and set up the application
inside the container.
::
    FROM node:10
    
    # Create app directory
    WORKDIR /usr/src/app
    
    # Install app dependencies
    # A wildcard is used to ensure both package.json AND package-lock.json are copied
    # where available (npm@5+)
    COPY package*.json ./
    
    RUN npm install
    # If you are building your code for production
    # RUN npm ci --only=production
    
    # Bundle app source
    COPY . .
    
    EXPOSE 8080
    CMD [ "node", "start" ]

Next, create the docker-compose file that will manage our application and
additional microservices.
::
    version: '3.3'
    
    services:
      web:
        build: .
        image: node:10
        ports:
          - '8080:8080'
        # restart: always
        volumes: 
          - ./:/usr/src/app
        networks:
          - restweb
    networks:
      restweb:

However, if you run the docker-compose up -d command, the container will not
run. If you run docker-compose logs, you'll notice that it can't find the
express npm package. Since mount the current directory into the container
at /usr/src/app, it overrides the directory and removes the node_modules
folder; therefore, the express package no longer exists. To counter this,
we will create an entry point file that runs the npm install command,
which creates the node_modules folder and other related files.
Lets called file entrypoint.sh.
::
    cd /usr/src/app
    npm install
    npm start

Afterwards, edit the Dockerfile to use the entrypoint.sh as the 
to run upon running a container.
::
    FROM node:10
    
    # Create app directory
    WORKDIR /usr/src/app
    
    # Install app dependencies
    # A wildcard is used to ensure both package.json AND package-lock.json are copied
    # where available (npm@5+)
    COPY package*.json ./
    
    RUN npm install
    # If you are building your code for production
    # RUN npm ci --only=production
    
    # Bundle app source
    COPY . .
    RUN mv entrypoint.sh /appstart.sh
    RUN chmod 744 /appstart.sh
    
    EXPOSE 8080
    CMD [ "/appstart.sh" ]


Creating a New Food Order
-----------------------------
Creating a new food order can be likened to sending a post request to an API,
the http post method allows you to send data from the client to the API.

First we need a variable to hold all the orders. Normally,
this would be a database (SQL, MongoDb),
But we are focusing on the developing the API application inside of the container
so we’ll skip the database layer.

Let’s create a variable to hold all the orders in our app.js file, like so:
::
    const orders = [];

This will hold all food orders that come into our API from the client.
To create a new food order, we’ll collect the following:

- Food Name
- Customer Name
- Food Quantity

To create a new order, input to the following code just before the app.listen,
like so::

    app.post("/new_order", (req, res) => {
      const order = req.body;
    
      if (order.food_name && order.customer_name && order.food_qty) {
        orders.push({
          ...order,
    
          id: `${orders.length + 1}`,
    
          date: Date.now().toString()
        });
    
        res.status(200).json({
          message: "Order created successfully"
        });
      } else {
        res.status(401).json({
          message: "Invalid Order creation"
        });
      }
    });

We created a new route, /new_order, with the post method to accept the incoming
food order data, and we check if any of the required data needed to created a
new order is valid, then we push a new order object to the the orders array we
created earlier and we add an id and date key to the order. To test restart
the server.

Now, lets test the creation endpoint with the command below:
::
    curl -X "POST" -H "Content-Type: application/json" -d '{"food_name": "chicken", "customer_name": "oshane", "food_qty": "2"}' "localhost:8080/new_order"

Getting All Food Orders
--------------------------
To retrieve all the food orders, simply input the code below, like so::

    app.get("/get_orders", (req, res) => {
      res.status(200).send(orders);
    });

This creates a get request on the /get_orders route and sends the orders
as an array to the client.

Updating a Food Order
-------------------------
To update a food order, we use the patch method, like so::

    app.patch("/order/:id", (req, res) => {
      const order_id = req.params.id;
    
      const order_update = req.body;
    
      for (let order of orders) {
        if (order.id == order_id) {
          if (order_update.food_name != null || undefined)
            order.food_name = order_update.food_name;
    
          if (order_update.food_qty != null || undefined)
            order.food_qty = order_update.food_qty;
    
          if (order_update.customer_name != null || undefined)
            order.customer_name = order_update.customer_name;
    
          return res
            .status(200)
            .json({ message: "Updated Succesfully", data: order });
        }
      }
    
      res.status(404).json({ message: "Invalid Order Id" });
    });

This accepts the order id and the updates related to the order id and updates
it by looping through the orders array to get the id that matches with the
one in the orders array and updates it. If it doesn’t find an order with the
id passed in to the route it returns a 404 http code and a message of
Invalid Order Id.

Deleting a Food Order
---------------------------
To delete a food order we create a route that accepts the id of the particular
order that needs to be deleted, like so::

    app.delete("/order/:id", (req, res) => {
      const order_id = req.params.id;
    
      for (let order of orders) {
        if (order.id == order_id) {
          orders.splice(orders.indexOf(order), 1);
    
          return res.status(200).json({
            message: "Deleted Successfully"
          });
        }
      }
    
      res.status(404).json({ message: "Invalid Order Id" });
    });

This accepts the order id and finds the order in the orders array by looping
current and getting the corresponding order with the id provided.
Using the splice method we can remove the order from the orders array.

With the tools and methods covered in this tutorial,
you should now be able to create simple REST APIs in Node.js using
Express in a containerized environment.
This is only an example of what can be done and you can extend this by 
connecting to a database, to make the data persistent.