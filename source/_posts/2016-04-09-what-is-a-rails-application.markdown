---
layout: post
title: "What is a Rails application"
date: 2016-04-09 14:47:14 -0700
comments: true
categories: [web development, web application, server, heroku, ruby on rails, rack, thin, deployment, nginx]
---

# What is a server and how does it relate to my Rails app?

I learned the basics of Ruby on Rails web application development a few years
ago. At that time, my understanding how that system works was as follows

1. Find articles describing things Rails allows you to do easily
2. Decide on an app that requires only those things
3. Follow the instructions in those articles to implement your app idea
4. Push the app to Heroku
5. Now the app is accessible to anyone on the World Wide Web

The fact that it is _so_ easy to do such a thing is nothing short of magical.
Especially while you have _no idea_ how any of it works. Now that I am
slightly more knowledgeable about how software systems are put together and
deployed, I'd like to take a slightly more nuanced stab at what really was
going on when the above 5 steps were executed.

<!-- more -->

## How Rails makes things so easy

Ruby on Rails was created out of frustration with "heavyweight" web
application development frameworks, such as ones from the late 90's that use
Java. It gives you base-classes for all the important parts of your program so
that you only need to write (and even _see_) code for the pieces that are
unique to your system. It generates a base project with everything you need
already included in a well-structured layout that helps you make sense of what
is going on. Through some clever usage of the Ruby programming language, it
creates methods named after the components you implemented, and makes them
available in the places you may need them. This has led to a culture of
natural-language inspired community-built APIs that enable you to easily do
things only the most modern websites have.

## How does traffic get into and out of Rails?

Maybe there is another way to do it, but at least by default Rails uses Rack.
Rack is a "webserver interface". A Rack application has a Ruby object that has
a method that receives parsed HTTP requests in a well-specified format, and
then calls a method (called `call`) which receives the parsed HTTP request as
a Ruby object, and returns a parsed HTTP response in a well-specified format
(specifically a Ruby array structured like `[response_code, {headers_map},
[body_lines]]`).

A Rails application implements Rack's `call` method. What that `call` method
actually returns for any given HTTP request argument is defined by all your
Rails code, e.g. your router, controllers, models, views, etc.

### What about Rack then?

Fine, now we understand that Rack passes the HTTP request to Rails, and Rails
returns the HTTP response to Rack. But how did Rack get the HTTP request in
the first place? Answer: from a Web Server.

A web server suitable for Rack would be a program that

* listens on a TCP port
* connects to clients on that port
* receives the HTTP requests from those clients
* parses them into the format that Rack expects
* Calls the Rack method `call` using that format
* Receives the HTTP response from Rack
* Converts that into the raw HTTP response format
* Sends the response to the client

There are many such suitable web servers, such as Unicorn and Thin.

### So what is nginx?

It seems to me that one does not typically use nginx to call Rack's `call`
method (though this might be possible to do). One typically uses nginx as a
waypoint of traffic bound for the application, but in need of some sort of
pre-processing. That pre-processing could take multiple forms. Maybe there is
a cache sitting outside of Rails itself, and nginx can be setup to return
results from the cache instead of proxying requests into the server which
interacts with Rails itself. This will improve performance for each client who
hits such a cache, and will increase the load the system is capable of
supporting overall. Nginx might know of multiple servers connected to
different instances of the Rails app running concurrently, and will choose
which server to send each request to in such a way as to distribute load
across the Rails applications ("load balancing"). Nginx might "terminate SSL",
meaning that clients connect to the address on which nginx is listening, and
will send their request to nginx over the TLS protocol, but nginx will decrypt
that request before passing it wherever it will go next; then nginx will
encrypt the response according to the TLS state and the client will know how
to decrypt the response (and ideally no one else will know how to decrypt it).

## What Heroku does

Using the simple `git push heroku` workflow, Heroku takes care of all the
aspects of managing the deployment of your Rails server. It will provide a
virtual server to run your Rails code. It will provide a server which speaks
Rack to receive requests, invoke your Rails code, and return the response to
the client. It has DNS entries installed in the relevant places on the
internet so that you can choose a name for your app to be accessed by your
friends in their homes using their browsers. It will do a whole lot more, such
as autoscaling, caching, and other things beyond the scope of this post
because I don't know about them.
