# Webbyduino

Arduino webserver with support for multiple WebSockets, authentication, cookies, protected pages/commands and multiple users.

## Features
- Modular design: only enable the features you need, making the library faster and smaller.
- Supports up to 4 simultaneous websockets (the maximum the ethernet shield can handle).
- Supports the following HTTP Methods: GET, HEAD, POST, PUT, DELETE, PATCH
- Parses GET, POST and Cookies data from the requests
- Supports login by HTTP Basic Authentication or by custom Cookies (automatic)
- Creation of JSON/RESTful interfaces
- Serve files from the SD card, automatically parsing them and replacing variables by their values if necessary.

## Demo Sketches

The best way to understand how this library works is by trying out the different sample sketches. I would suggest checking them in this order (increasing complexity):

- WebbyduinoHello - "Hello Word" version of the server, simply creates an HTML page that says "Hello World".

- WebbyduinoReadAnalogSD - Open a page with the values from the analog pins. The page is stored on the SD card and the values of the readings are replaced every time the page is accessed.

- WebbyduinoCookies - A server that remembers your name by using cookies. The first time you connect it asks for your name and stores it in a cookie. Once the cookie is set, it always greets you using your name.

- WebbyduinoSecurity - A server with a few sample pages with different access level restrictions. Different user logins are necessary to access the different pages.

- WebbyduinoChat - A chat server where multiple users can connect simultaneously and talk to each other in a common chatroom. It uses Websockets for the communication between users once they are connected.

- WebbyduinoHTTPServer - HTTP server running from the SD card. Acessing the root of the server opens the file "index.htm" from the card.

- WebbyduinoFileBrowser - Similar to the HTTP Server, but allows navigating through all the existing directories on the card and downloading every file.

- WebbyduinoControlIO - Control the state of output pins from the browser, and also see the state of the analog pins changing in real-time. Every update is broadcasted to all connected clients using websockets.

- WebbyduinoLedRGB - Control a RGB LED from the browser using 3 sliders for the different colors. All the sliders on every connected client are updated whenever anything changes in one of them.
 
- WebbyduinoLedRGB_SD - Same as WebbyduinoLedRGB, but the web page is stored on the SD card instead of the Arduino memory. This is more convenient when having large pages/scripts.


### Memory issues
This library offers a lot of features, which unfortunatelly all add up when considering memory usage. It is possible to reduce the memory usage by removing the unwanted features from the code. This can be done simply by removing specific #defines in the beginning of the "Webbyduino.h" file. By default, all the features on the library are enabled, just comment the ones not in use:

    // Enable support for GET parameters
    #define WEBBYDUINO_ENABLE_GET
    
    // Enable support for POST parameters
    #define WEBBYDUINO_ENABLE_POST
    
    // Enable websockets functionality
    #define WEBBYDUINO_ENABLE_WEBSOCKETS
    
    // Enable parsing parameters on received websocket messages
    // (assumes websocket messages follow the POST/GET parameter syntax)
    #define WEBBYDUINO_ENABLE_WEBSOCKETS_PARAMETERS
    
    // Enable support for Cookies
    #define WEBBYDUINO_ENABLE_COOKIES
    
    // Enable support for users/security
    #define WEBBYDUINO_ENABLE_USERS
    
    // Enables login functionality using Basic Authentication
    #define WEBBYDUINO_ENABLE_AUTHENTICATION_BASIC
     
    // Enables (automatic) login functionality using cookies and POST data 
    #define WEBBYDUINO_ENABLE_AUTHENTICATION_COOKIES
    
    // Enables serving files from the SD card
    #define WEBBYDUINO_ENABLE_SD_CARD

Additionally, when not using the SD card the line
    
    #include "SD.h"
    
should also be removed from the main sketch file.
    

## Getting started

### Create your server

To create a server, initialize the ethernet normally and create the server object. The server can be initialized with 3 optional parameters:

    // Default initialization
    Webbyduino webserver;  
    // Custom initialization
    // Webbyduino(const char *homepage = "/", int port = 80, byte maximum_websockets = 3);
    Webbyduino( "/home/", 88, 2);

With the custom initialization we can override the settings for the root directory of the site, the port used for the server, and the maximum number of simultaneous websockets allowed.


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

##### Acessing GET/POST/Cookies data 
    
If we want to use GET/POST/Cookies data we can access it inside this function.
This example demonstrates how to read Cookie parameters. It's exactly the same for POST/GET parameters, we only change readCookieP() to readPOSTParameterP() or readGETParameterP(). In this case we are only interested in reading the name of a cookie called "user_name" and copy its value to a buffer, so the page will say "Hello usernamegoeshere" instead of "Hello World":

    int myHomepage(Webbyduino &server)
    {
        // Create buffers to store the data we're interested
        #define NAME_MAX_SIZE 32    
        char user_name[NAME_MAX_SIZE] = "";
    
        // Reads the value of the cookie "user_name" to the buffer
        // If the cookie does not exist,  user_name remains unchanged (empty)
        server.readCookieP(PSTR("user_name"), user_name, NAME_MAX_SIZE)

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
            
##### Sending data from a SD file

Another interesting option, specially if the HTML pages are too big (or too many) to fit in the Arduino memory is to read them directlly from files on the SD card. Webbyduino can do this automatically, and also replace parts of the file with variables of the program. The next example does exactly the same as the previous one, but it reads the HTML from a file, replacing the user name with the value read from the cookie:

    int myHomepage(Webbyduino &server)
    {
        // Reading the cookie is exactly the same as before
        #define NAME_MAX_SIZE 32    
        char user_name[NAME_MAX_SIZE] = "";
        server.readCookieP(PSTR("user_name"), user_name, NAME_MAX_SIZE);
        
        // This line works similar to a #define in C.
        // When sending the next file, all instances of %USER_NAME will be replaced
        // with the value of the variable user_name
        server.addSDFileParameterP(PSTR("USER_NAME"), user_name );
      
        // Finally we just need to indicate the file we want to send. The content of
        // the file is almost the same as in the previous example, we only need to 
        // indicate where the user name will appear. This is donw by simply writting
        // "Hello %USER_NAME" so the parser knows where to replace the variable.
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
        // This function receives a buffer with the data, its length and the socket that sent it
        // Prints the string received from the websocket to the console
        Serial.print(F("onWebsocketData: ")); Serial.println(data_string);
        
        // This line would broadcast the received message to every connected websocket
        server.websocketBroadcast(data_string);
        
        // This variation would broadcast to everyone except the one the sent the original message
        server.websocketBroadcast(data_string, socket);
        
        // This line would send the received message to a specific websocket number (2)
        server.websocketSendClient(data_string, 2);


        // This line would close the websocket that sent the message, if the message was "CLOSE"
       if (strcmp_P(data_string, PSTR("CLOSE")) == 0) void websocketClose(socket);
       
       // Websocket data can be automatically processed and acessed the same was as  
       // GET/POST/Cookies data. If the browser sends a websocket message like this:
       // "red=12&green=28&blue=0" it is parsed and the data cab be accessed like this:
       
       if (server.readWebsocketParameterP(PSTR("red"), value, VALUE_BUFFER))   
       {
           // if the parameter "red" exists, do something with the value
       }
       
    }

We can send websocket messages anytime, not just when the browser sends something. We can send strings with data instantly, or append them to a buffer to create a more complex message:

      // Construct the message to send
      webserver.wSend("a0=");
      webserver.wSend(analogRead(0));
      webserver.wSend("&a1=");
      webserver.wSend(analogRead(1));

      // Calling websocketSendClient or websocketBroadcast without a buffer sends 
      // the message stored in the buffer constructed with the wSend methods.
      // This would send the message "a0=123&a1=234" to everyone and clear the buffer
      webserver.websocketBroadcast();
    

### Using Cookies

Cookies are used to store data on each browser. The server sends the cookies to the browser when it replies to an HTTP request. We add the cookies when processing the data in the callback functions for the commands:

    // Add cookies to the replies using lines like these
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


## Compatibility

This library was tested with Arduino Ethernet, Uno and Mega with Ethernet shields. It should be compatible with any arduino clone that has at least 32Kb of memory and uses the Wiznet 5100 chip for the ethernet.

## Version History

### 1.0 released in 2014-06-? (soon-ish :P)
