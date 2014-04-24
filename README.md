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

Documentation
========
* [Installation](http://github.com/stoffera/duality/wiki/installation)
* [Examples](http://github.com/stoffera/duality/wiki/examples)
* [Getting Started](http://github.com/stoffera/duality/wiki/getting-started)
    * [Routes](http://github.com/stoffera/duality/wiki/routes)
    * [Serving the content of a file](http://github.com/stoffera/duality/wiki/serving-a-file)
    * [Sessions](http://github.com/stoffera/duality/wiki/sessions)
    * [Access Control](http://github.com/stoffera/duality/wiki/access-control)
* [API Reference](http://github.com/stoffera/duality/wiki/api-reference)

License
=======
Duality is licensed under GNU Lesser Public License v2.1. The license header is found in every source file.
Please refer to the [LGPL](http://www.gnu.org/licenses/lgpl-2.1.html) site for more
