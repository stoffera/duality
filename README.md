Duality Web Server
=======

A simple web server, servering static files and binding URLs to handler functions

This is a simple [node.js](www.nodejs.org) HTTP server. It can only serve content in 2 ways, hence the name *Duality*.  
These ways are:

1. It can serve static files from a root directory, just as any other conventional server.
2. It can map URL's to specific functions and pass specific arguments.

**This simple approach of serving content is very powerful.**

Serve your **HTML**, **JS**, **CSS**, etc. through method no. 1.  
Then use method no. 2 to provide you WebApp with specific processed data.

Features
========

Duality has the simplicity of the standard node.js HTTP Server. In fact, it is build on top of it.  
But more important it gives you convenient extra functionalities not found in node's standard HTTP server.

* **Partial Content**. Clients can request only a specific range from a file. This is by default supported.
* **Session support**. Just set the `useSessions` flag, and the server tracks every request with sessions.
* **Access Control**. Turn on access control and provide a callback function that gets called on every access request. This allows easy and fine grain control of which user (session) has access to which resource.
* **Authentication**. Provide a login redirect URL or use the build in HTTP Authentication. (The digest variant)
* **Caching** Duality supports the HTTP cache scheme, where only changed are transmitted. Bandwidth is not wasted on files not changed since last request.
* **Access logging**. If you wish, every access can be written to stdout.

Along with these features comes a lot of handy options you can control. Such as the server identification header and the sessions cookie name.

Index
========
1. [Features](#features)
* [Installation](#installation)
* [Documentation](#documentation)
	1. [Serve static files](#a-quick-example)
	* [Routes](#defining-routes)
	* [Serving content of a file](#serving-content-of-a-file)
	* [Sessions](#sessions)
	* [Access Control](#access-control)
		1. [Login method](#login-method)
		* [Callbacks](#callbacks)
* [API Reference](#api-reference)
	1. [Object Methods](#object-methods)

Installation
============
Download the `duality.js` file and place it in the working directory of you node.js project. Include the duality module in the file that should use the server:

```javascript
var duality = require('./duality.js');
```

Now the `duality` is defined in the global context. See the [API Reference](#api-reference) for the list of defined functions.

Documentation
=============
It is easy to setup Duality for whatever web service you need. Let first see:

####A quick example
A simple static server, that just server files from a directory:

```javascript
	var duality = require('duality');
	var server = duality.createServer("/var/www");
```

And that is it. A duality server instance now serves every file found inside `/var/www/`. Note that directory listing is not supported by duality at this point.

Lets say you are hosting a blog through the duality server. All static files like html, js, css and images gets served by the static method. Now functions for returning a list of posts and individual posts are needed. To do this duality can bind specific URLs to function callbacks. This is done by:

####Defining routes
The routes table is a JSON object that bind regular expressions to functions. The keys in the object are strings written as regular expressions, and the values are functions. (Note that object key can only by of type: `string`.) If any incoming requests URL matches a regular expression in the routes table, the related function is called.

We define a regexp for extracting a list of posts in a blog, like this: `"^/posts/?$"`. This means any URL of the form: `/posts` or `/posts/` will match the regex. We define a function to extract all posts from a data store:

```javascript
	var getAllPosts = function(match, req, res) {
		
		/* Code to extract post data */
		
		/* Our function must return data to the client */
		res.writeHead(200, {'Content-Type':'application/json'});
		res.end(json_string);
	};
```

The defined function is now mapped to a URL in the a route table:

```javascript
	var routes = {
		"^/posts/?$" : getAllPosts
	};
	
	/* create and start the server */
	var server = duality.createServer("/var/www",routes);
```

Now Duality will automatically call `getAllPosts` everytime the request URL matches the regexp. The three arguments passed to the function are a regular expression `match` object, and node.js' `http.ServerRequest` and `http.ServerResponse` objects.

Next we want to get individual blog posts in a RESTful manner like: `/posts/[id]`. To do this the function called from the routing table must receive parameters extracted from the regexp. Extracting parameters is done with parenthesis in the regexp, and by quering the `match` object. (Please refer to the [JS docs](http://www.w3schools.com/jsref/jsref_match.asp) on how to do this.) First we add the new function to the routing table:

```javascript
	var routes = {
		"^/posts/?$" : getAllPosts,
		"^/posts/(\\n+)" : getPost
	};
	
	/* create and start the server */
	var server = duality.createServer("/var/www",routes);
```

First notice the double escaped backslashes. Because Duality will take the string and create a JS `RegExp` object from it, it is parsed twice. First when the JS interpreter read your code, second when the `RegExp` object initializes from the string. This means you **must** double escape you characters for the route to work properly.

Now to extract the post id from the URL, we have added the parenthesis around the numeric character wildcard. This will pass the post id to the `getPost` function as a part of the `match` object. In the function it is easily extracted:

```javascript
	var getPost = function(match, req, res) {
		var id = match[1];
		
		/* get the post data for the id */
		
		/* return the post data to the client */
		res.writeHead(200,{'Content-Type':'application/json'});
		res.end(json_string);
	};
```

Now we can serve blog posts from functions and static files directly from the disk. But often you will need to send the content of a file to the client, from within a route function. Lets look at that next:

#### Serving content of a file
If you are inside a function called by a route, you may need to serve an entire file. Duality provide the method `serveFile` for this on the instantiated server object. In fact this the very method used by Duality to serve all static files.

`serveFile` takes 5 arguments:

* **filePath** the path to file that is to be served. Relative or absolute. All files are served, with no regard to type or size.
* **request** The nodeJS `http.ServerRequest` object.
* **response** The nodeJS `http.ServerResponse` object.
* **dont\_detect\_MIME\_type** An optional switch to disable automatic MIME type detection, based on the file name extension. If you set this to false you must insert the `Content-Type` header into the response yourself. This can be done before to call `serveFile`.
* **attachment** An optional switch to tell the client browser that the file should be downloaded - not displayed inside the browser.

`serveFile` does not return any value, and works asynchronous. After calling the function on the server object instance, you should not touch the `http.ServerResponse` object. Here is a example on how the function works:

```javascript
	var getCachedPost = function(match, req, res) {
		/* get the path of the file containing the cached post */
		var path = getTheCachePath( ... );
		
		/* Serve the file, forcing the MIME type to JSON */
		res.setHeader('Content-Type','application/json');
		server.serveFile(path, req, res, true);
	};
	
	/* route setup and definitions are omitted */
	
	/* start the server */
	var server = duality.createServer("/var/www", routes);
```

The `serveFile` method work well with large files and ranges of files. Clients can use HTTP headers to only query a specific portion of the file. The `serveFile` handles this case by only serving the requested file portion.

Furthermore `serveFile` implements caching of responses. The clients can include headers to tell they only want the file, if it has changed since last visit. This is handled by returning the *Not Modified* response.

### Sessions
To be written...

###Access Control
To be written...
####Login Method

####Callbacks

API Reference
=============

To create a server use the function `createServer()`, this will create an instance of the duality server object.

####`createServer( server_directory [, routes, options] )`
Return a new server object, serving the folder found at `server_directory`. Every file in this directory is served.
If you defined any routes in the optional `routes` object, then these functions will be called when the request URL match.
If you just want a static file server, then omit the routes table parameter. The server takes an options object. You can set additional server parameters through this object.

* **server_directory** *string* The path to the folder where the static files are shared from
* **routes** *Object* (OPTIONAL) A dictionary of regex string and corrosponding functions.
* **options** *Object* (OPTIONAL) A dictionary of extra options to the server
* **return** *duality* An instance of the duality server class.

##### Routes
If you what duality to bind a function to a URL you must provide a route. A route is a regexp formatted string and a function. The concept is properly best explained with an exmaple:

```javascript
	var routes = {
		"^/posts/(\\n+)" : getPost,
		"^/posts/?$" : getPostIndex
	};
```

Duality will parse each string into a regular expression and check all request URL's against them. Notice how the backslash is double escaped. You must do this because the string is parsed into a regexp, causing the escape characters to be parsed twice. The functions related to the regexp's receive three arguments: a basic java script `string.match` object, a `http.ServerRequest` object and a `http.ServerResponse` object. If you define regions with parentheses in the regexp, then these will available in the `match` object. In this way you can pass parameters to the function, from the parsed URL.

#####Options Available
Here is the list of optional options to pass to the server:

* **serverPort** *number* The listening port of the server. Multiple server instances can listen on different ports. Default: 8080
* **useSessions** *boolean* Enable sessions on the server. Each request will be identified with a session. Default: TRUE
* **sessionIdentifier** *string* The name of the cookie in the client, which stores the session unique value. If sessions are disabled, this has no effect. Default: `duality-session`
* **serverString** *string* The value of the `Server` HTTP header, identifying the server for the client. Default: `duality`
* **useAccessLog** *boolean* Enable the stdout access log. Every request will be logged. Default: TRUE
* **useAccessControl** *boolean* Activate the access control system. See documentation on how to use access control. Default: FALSE
* **accessLoginUrl** *string*  URL to a login page or a function to display the login page. If NULL and access control system is activated - HTTP Auth is used. Default: NULL
* **accessResourceCallback** *function* Resource access control callback function. See access control documentation. Default: NULL
* **httpAuthRealm** *string* The http authentication realm. Default: `Login`
* **httpAuthUserLookupCallback** *function* Callback function you must define to check the HTTP Auth login credentials. Default: NULL
* **httpAuthLoginSuccessCallback** *function* Callback to tell a HTTP Auth login succeeded. Default: NULL
* **httpAuthLoginFailedCallback** *function* Callback to tell a HTTP Auth login failed. Default: NULL

####`getHttpAuthUserDigest( user, realm, password )`
Global function to create the HTTP Auth Digest HASH of Username:Realm:password if you know what Realm to use, then this function can be used to create hashes to store in a user table. This will eliminate the need to store the password in clear text.

* **username** *string* The username, which is a part of the hash
* **realm** *string* The realm used for the HTTP Auth
* **password** *string* the password in clear text
* **return** *string* The hashed value as a HEX string

***

## Object Methods
Once the server object is created, these methods can be called on the object.

####`serveFile( path , req, res [, dont_detect_content_type, attachment])`
Serve the content of any file on the disk. This method is used by the internal static files serving. You can also call this function from you own code to serve any file you like. Partial content ranges are supported, so only part of the file is served, if request by the client.

* **filePath** *string* path to the file that should be served
* **req** *http.ServerRequest* the current http request
* **res** *http.ServerResponse*  the response for the current http request
* **opt\_dont\_detect\_content\_type** *boolean*  (OPTIONAL) Set this to true to avoid auto detection and setting of content-type header
* **opt_attachment** *boolean* (OPTIONAL) Set this to TRUE to tell the browser to serve the file as a download

#### `httpAuthHash( username, password )`
Create a hash to use with the HTTP Authentication Digest for the server. If you maintain a list of users and their passwords (in clear text), then use this method inside the `httpAuthUserLookupCallback` function to return a valid hash for the HTTP Auth digest. (Not recommended)

A way better approach is to pre-hash the users password, either with this method or `getHttpAuthUserDigest()`. Then only store the hashed password in your users table.

* **username** *string* The username is needed as part of the hash in HTTP Auth Digest
* **password** *string* The password in clear text
* **return** *string* A HEX encoded hash string