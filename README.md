# Webbyduino

Arduino webserver with support for multiple WebSockets, authentication, cookies, protected pages/commands and multiple users.

## Features
- Modular design: only enable the features you need, making the library faster and smaller.
- Supports up to 4 simultaneous websockets (the maximum the ethernet shield can handle).
- Supports the following HTTP Methods: GET, HEAD, POST, PUT, DELETE, PATCH
- Parses GET, POST and Cookies data from the requests
- Supports login by HTTP Basic Authentication or by custom Cookies (automatic)
- Creation of JSON/RESTful interfaces


## Demo Sketches

The best way to understand how this library works is by trying out the different sample sketches that come with it:

- Hello Server - A "Hello Word" version of the server, simply creates an HTML page that says "Hello World".
- Sensor Server - A server that keeps broadcasting the values of different sensors (digital and analog readings from arduino pins in this case) and updating those values on the browser instantly. 
- Chat Server - A chat server where multiple users can connect simultaneously and talk to each other in a common chatroom. It uses Websockets for the communication between users.
- Memory Server - A server that remembers your name by using cookies. The first time you connect it asks for your name and stores it in a cookie. Once the cookie is set, it always greets you using your name.
- File server - A file server that allows the users to browse the contents of the SD card and download files from it.
- Secure server - A server that has different commands which require different user levels to be executed.


## Getting started

### Create your server

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

     
We can also set a default callback for when all other fail. This is useful specially to serve files from a SD card. We can have the commands we want using the callbacks above, and when the requests don«t match any of the known commands the default callback checks if it matches something on the SD card.

    webserver.setCommandDefault(&fileServerPage);
    
All it takes now is to call the processConnection() method often on the loop and the server is working!

        void loop()
        {
            // Process incoming connections to the webserver
            webserver.processConnection();
            
            // More code can be added here if necessary    
        }

### Creating callback functions

Callback functions are called when the server receives specific requests. They can be used to perform any task before sending a reply to the client that sent the request. When processing the callback functions we have access to all the data sent by the client.

Here is a sample of a callback function that responds with a simple HTML page saying "Hello World":

    int myHomepage(Webbyduino &server)
    {
        // Before sending the actual reply we need to send the headers indicating 
        // everything went ok and we are atarting to send the reply
        server.httpSuccess();   // No parameters assumes we're sending HTML
        // Now we send the actual HTML code for the response
        server.print( F(
            "<!DOCTYPE html>\n"
            "<html>\n"
            "<head>\n"
            "<meta http-equiv='Content-Type' content='text/html; charset=utf-8' />\n"
            "<title>Webbyduino Server</title>\n"
            "</head>\n"
            "<body>\n"
                "Hello World"));
            "</body>\n"
            "</html>\n" ));
        return 1;
    }
Return 1 if everything went well with the request.
Returning 0 results in a 404. After calling httpSuccess() always use return 1. All necessary validations that may result in an error (return 0) need to be checked before before calling httpSuccess().

    
If we want to parse GET/POST/Cookies data we can access it inside this function.
This example demonstrates how to read Cookie parameters. It's exactly the same for POST/GET parameters, we only change readCookieP() to readPOSTParameterP() or readGETParameterP(). In this case we are only interested in reading the name of a cookie called "user_name" and copy its value to a buffer, so the page will say "Hello usernamegoeshere" instead of "Hello World":

    int myHomepage(Webbyduino &server)
    {
        // Create buffers to store the data we're interested
        #define PARAMETER_VALUE_BUFFER 32    
        char user_name[PARAMETER_VALUE_BUFFER] = "";
    
        // Reads the value of the cookie "user_name" to the buffer
        // If the cookie does not exist,  user_name remains unchanged (empty)
        server.readCookieP(PSTR("user_name"), user_name, PARAMETER_VALUE_BUFFER)

        // This part is similar to the previous one, but we insert the user name in the page
        server.print( F(
            "<!DOCTYPE html>\n"
            "<html>\n"
            "<head>\n"
            "<meta http-equiv='Content-Type' content='text/html; charset=utf-8' />\n"
            "<title>Webbyduino Server</title>\n"
            "</head>\n"
            "<body>\n"
                "Hello "));
                
        // Prints either the deafult user name or the value read previously from the cookie
        server.print(user_name);
        
        // Prints the rest of the HTML page
        server.print( F(
            "</body>\n"
            "</html>\n" ));
        return 1;
    }
            

Another interesting option, specially if the HTML pages are too big (or too many) to fit in memory is to read them directlly from files on the SD card. Webbyduino can do this automatically, and also replace parts of the file with variables of the program. The next example does exactly the same example as the one above, but ir reads the HTML from a file, replacing the user name with the value read from the cookie:

    int myHomepage(Webbyduino &server)
    {
        // Reading the cookie is exactly the same as before
        #define PARAMETER_VALUE_BUFFER 32    
        char user_name[PARAMETER_VALUE_BUFFER] = "";
        server.readCookieP(PSTR("user_name"), user_name, PARAMETER_VALUE_BUFFER);
        
        // This line works similar to a #define in C.
        // When sending the next file, all instances of %USER_NAME will be replaced
        // with the value of the variable user_name
        server.addSDFileParameterP(PSTR("USER_NAME"), user_name );
      
        // Finally we just need to indicate the file we want to send. The content of
        // the file is almost the same as in the previous example, but we neem to write
        // "Hello %USER_NAME" so the value is replaced every time.
        return server.sendFileHTTP("test.htm");
    }
            


### Using Websockets

The Websocket requests are handled automatically by the server, as long as there are available sockets. The server only supports unfragmented text frames, meaning we can only send messages up to 125 chars long.

Callbacks can be added to perform actions when a websocket connects, when it disconnects or when data is received.
Example of the incoming data callback function:

    // Callback function for when a websocket receives data
    // This function receives a pointer to the data received on the message, and it's length,
    // as well as the number of the socket where the message was received
    void onWebsocketData(Webbyduino &server, char* data_string, byte frame_length, byte socket) 
    {  
        // In this function we can parse the incoming data and do whatever we want with it
        // Prints the string received from the websocket to the console
        Serial.print(F("onWebsocketData: ")); Serial.println(data_string);
        
        // This line would broadcast the received message to every connected websocket
        server.websocketBroadcast(data_string);
        
        // This line would send the received message to a specific websocket number
        server.websocketSendClient(data_string, 2);
        
        // This line would close the websocket that sent the message, if the message was "CLOSE"
       if (strcmp_P(data_string, PSTR("CLOSE")) == 0) void websocketClose(socket);
    }


### Using Cookies

Cookies are used to store data on each browser. The server sends the cookies to the browser when it replies to an HTTP request. We add the cookies when processing the data in the callback functions for the commands:

    // Add cookies to the replys using lines like these
    sendCookie("user_name", "Jane");
    
Any number of cookies can be added, and they are sent when server.httpSuccess() is called, so it's really importate to declare them all before that.

### Security

All the pages can be protected. The server supports different methods of authentication, either by Basic Auth or by Cookies. Note that it does not support data encryption, because the arduino does not have enough resources to handle it.

All the users have an assigned user level, so we can have different commands accessible to different users.

Add users to the server with the followint method:

    // Adds a new user to the server user list
    void addUser("jones", "pass123", 3, 2);

When creating a new user, we specify the user name (jones), password(pass123), the user level (8) and the maximum allowed websockets (2).

The user level is a value used for automatic access control on specific commands. To add secury to any command just add the minimum user level required for that command. Adapting the previous example,

    webserver.addCommand(PSTR("files"), fileServerPage);

would become

    webserver.addCommand(PSTR("files"), fileServerPage, 4);

meaning that only users with level 4 or more have access to this resource. User with a lower clearence level receive a 401 (Authorization Required) error page.

Alternativelly, the pages can be acessible to different user levels, but show different content or perform different actions depending on each level. On the callback functions that process the different commands, we can use the method

    server.getClientLevel();

to discover the user level of the client that performed the request and use that information when constructing the reply.

### Customize your server
This step is optional, but it's recommended once you're acquainted with the library. Since the arduino is very low on resources when compared to a PC, it's recommended to disable the features of the server that are not going to be used. This saves precious program memory, decreases RAM usage and makes the server process the requests faster. Edit the file "Webbyduino.h", look for the following lines and comment the ones related to features you do not want to use.

        // Removing this line disables all support for GET parameters
        #define WEBBYDUINO_ENABLE_GET
        
        // This line removes all support for POST parameters
        #define WEBBYDUINO_ENABLE_POST
        
        // This line removes all websocket functionality
        #define WEBBYDUINO_ENABLE_WEBSOCKETS 
        
        // This line removes all cookie functionality
        #define WEBBYDUINO_ENABLE_COOKIES
        
        // This line removes all user functionality
        #define WEBBYDUINO_ENABLE_USERS

## Compatibility

This library was tested with Arduino Ethernet, Uno and Mega with Ethernet shields. It should be compatible with any arduino clone that has at least 32Kb of memory and uses the Wiznet 5100 chip for the ethernet.

## Version History

### 1.0 released in 2014-06-? (soon :P)
