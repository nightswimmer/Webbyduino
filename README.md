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

## Compatibility

This library was tested with Arduino Ethernet, Uno and Mega with Ethernet shields. It should be compatible with any arduino clone that has at least 32Kb of memory and uses the Wiznet 5100 chip for the ethernet.

## Version History

### 1.0 released in 2014-06-? (soon :P)
