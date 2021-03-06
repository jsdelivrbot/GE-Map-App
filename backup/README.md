# Creating a server-side CRUD module using Express
In assignment 1, we created a simple node server that retrieved our customers by responding to GET requests to '/customers'. You are now going to add more functionality to this server that allows us to **create**, **read**, **update**, and **delete** customers from a Mongo database. These tasks are commonly referred to as CRUD. 

## Introduction to Express

While these new requests could be handled in the same fashion as the original request handler, it would quickly become unweildly. There would need to be a bunch of conditional statements to handle requests to the different URL paths and different HTTP methods (such as POST, PUT, and DELETE). Luckily, the [**Express**](http://expressjs.com/en/index.html) library makes this task much simpler by providing a layer of abstraction for handling HTTP requests in a Node server. 

To provide an example, here is the request handler we wrote in assignment 1:

```javascript
var requestHandler = function(request, response) {
  var parsedUrl = url.parse(request.url);

  if(request.method === 'GET') {
    if(parsedUrl.path === '/customers') {
      response.writeHead(200, { 'Content-Type': 'application/json' });
      response.end(JSON.stringify(listingData));
    } else {
      response.writeHead(404);
      response.end('Bad gateway error'); 
    }
  } else {
    response.writeHead(404);
    response.end('Bad gateway error');
  }
};
```

Now here is the same request handler written using Express:
```javascript
app.get('/customers', function(req, res) {
  res.send(listingData);
});

app.all('/*', function(req, res) {
  res.status(404).send('Bad gateway error');
});
```

## Middleware
Understanding the concept of **middleware** is extremely important in using Express effectively. Middleware allows you to invoke functions on a request before it reaches its final request handler. As a simple (yet quite useless) example, let's add a greeting to each request made to the server. 

```javascript
app.use(function(req, res, next) {
  req.greeting = 'Hello there!';
  next();
});

app.get('/', function(req, res) {
  res.send(req.greeting);
});
```
In addition to the usual request and response objects, we now pass an additional object called *next*. Invoking *next* will pass the request on to whatever function is next in line to handle it. 

Now, let's say the application we are building has users with administrative privledges. There will be certain routes that we want to make sure the user has the correct privledges before allowing the request to be handled. Using express, this becomes a relatively simple task:

```javascript
var checkPermissions = function(req, res, next) {
  if(req.isAdmin === true) {
    next();
  } else {
    res.status(400).send('User does not have permission to access this path');
  }
};

app.get('/privateData', checkPermissions, function(req, res) {
  res.send('Some really critical information');
});
```

The checkPermissions function serves as *middleware* that is invoked before passing the request to its final destination. 

A final note: **order matters** when using middleware. If you place `app.use()` after a request handler, that middleware will not be invoked. Keep this in mind when developing your applications in case you encounter bugs. 

If the concept of middleware is still confusing, you can read [this blog post](https://www.safaribooksonline.com/blog/2014/03/10/express-js-middleware-demystified/) for further information. 

## Assignment Details
Now go ahead and clone this assignment's repository. You'll notice that the file structure of the application is now more involved than previous assignments. Browse around and take note of where each part of the application exists. 

Navigate to `server/config/express.js`. This is where you will place code to configure your Express application. The **morgan** module is used to log requests to the console for debugging purposes. The **body parser** module is middleware that will allow you to access any data sent in requests as `req.body`. 

In `server/routes/customers.server.routes.js`, you will find code that specifies the request handlers for CRUD tasks. To learn more about the Express router, [go to this page](http://expressjs.com/en/guide/routing.html) and scroll down to the section on *express.Router.*

### Part 1
Create a diagram of how the different parts of the server interact with one another. Specifially make note of: 
   - what is defined in the controllers
   - how the router makes use of the controllers to determine the flow of request handling
   - how middleware is used throughout the application to modularize the code


### Part 2

1. Implement the request handlers in `customers.server.controller.js`
    - test your implementation by running the tests found in `customers.server.routes.test.js`
2. Complete the app configuration in `express.js`. 
    - serve the static files found in the public folder when a user makes a request to the path `/`. [Refer to this documentation](http://expressjs.com/en/starter/static-files.html) for help
    - use the customers router for requests going to the `/api/customers` path 
    - direct users to the client side `index.html` file for requests to any other path
3. Make sure your server is functioning correctly by starting it up by running the command `node server.js`

