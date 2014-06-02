# Webbyduino

Arduino webserver with support for multiple WebSockets, authentication, cookies, protected pages/commands and multiple users.

## Features
- Modular design: only enable the features you need, making the library faster and smaller.
- Supports up to 4 simultaneous websockets (the maximum the ethernet shield can handle).
- Parses GET, POST and Cookies data from the requests
- Supports the following HTTP Methods: GET, HEAD, POST, PUT, DELETE, PATCH
- Supports login by HTTP Basic Authentication or by custom Cookies (automatic)
- Creation of JSON/RESTful interfaces


## Demo Sketches

There will be videos demonstrating the Sample sketches that come with the library.

- [Chat Server](http://youtube.com)
- [File Server](http://youtube.com)
- more to come

## How to use

This quick guide will try to explain most of the features of this library.

under construction...



### Getting started

To create a server, initialize the ethernet normally and create the server object. The server can be initialized with 3 optionl parameters:

    // Default initialization
    Webbyduino webserver;  
    // Custom Initialization
    Webbyduino(const char *homepage = "/", int port = 80, byte maximum_websockets = 3);

With the custom initialization we can specify the root directory of the site, the port used for the server, and the maximum number of simultaneous websockets allowed.


In the setup() we need to specify the pages/commands we want to serve

We should specify what the server opens when we access the root of the site:

    webserver.setCommandDefault(&myHomepage);

myHomepage if a callback for whatever we want to do when someone accesses the root of the site. In this callback function we can access received POST, GET and Cookie data from the request, and decide how to reply.

If we want to add more commands, just use the method below and add more callback functions.

    webserver.addCommand(PSTR("files"), fileServerPage);
    
We can also set a default callback for when all other fail. This is useful specially to serve files from a SD card. We can have the commands we want using the callbacks above, and when the requests don«t match any of the known commands the default callback checks if it matches something on the SD card.

    webserver.setCommandDefault(&fileServerPage);
    
All it takes now is to call the processConnection() method often on the loop and the server is working!

### Creating callback functions

under construction...

### Using Websockets

under construction...

### Reading POST/GET parameters

under construction...

### Using Cookies

under construction...

### Security

under construction...

## Compatibility

This library was tested with Arduino Ethernet, Uno and Mega with Ethernet shields. It should be compatible with any arduino clone that has at least 32Kb of memory and uses the Wiznet 5100 chip for the ethernet.

## Version History

### 1.0 released in 2014-06-? (soon :P)
