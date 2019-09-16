---
layout: post
title: How to: set-up a simple API with Clojure
date: 2019-09-16
Author: MatthewLisp
tags: [learn]
comments: true
toc: true
pinned: false
---
## Introduction
**What this guide will cover?**<br/>
This guide aims to set you up to start writing your own API using Clojure. We will not write an entire api, here i'll just show how you set everything up in order to start writing yourself.

**Who is this guide for?**<br/>
It's aimed mainly for experienced developers who is coming to clojure but don't yet know the *Clojure way* to create a simple API. I wont teach you how to create an api from the zero, nor the specific things that you need to understand in order to create an api, such as http requests, if you never created an api or don't know how it works, wrong guide for you. But if you're looking to fast get your head around the clojure way, that's the place.

**What exactly we are going to do?**<br/>
Using a set of libraries, we will bootstrap the code for an API in clojure, we're going to see how we deal with HTTP requests and responses, how we parse them to JSON and how to do the Routing of incoming requests.

**Is this the correct way of building an API in clojure?**<br/>
The short answer is always: *It depends*.<br/>
It depends on your project needs, and this is not the only way to create an api in clojure and it's absolutely not the standard way, because in clojure we have no standards for how you should build your systems. It's just a simple and fast way to understand how to start messing around with it.<br/>

You're going to see that the way we are doing is very manual and you have to set-up a lot of stuff to start, but this *way* of doing it and the libraries used here, is the very foundation for most of clojure frameworks i've seen, therefore if you truly understand this guide, youl'll end up understanding lots of stuff related to clojure for the web.

**What you will need**<br/>
* Basic understanding of clojure syntax
* Clojure data structures 
* Functional programming fundamentals 
* [How clojure namespaces works](https://www.braveclojure.com/organization/)<br/>
* Make sure you have [Leiningen](https://leiningen.org/) installed and off course [Clojure](https://clojure.org/guides/getting_started)<br/>

**What if i don't have yet what's needed?**<br/>
I recommend you take a look on the resources i've curated to learn clojure and at least understand the language core aspects: https://github.com/matthewlisp/learn-clojure
 
***I strongly suggest that you follow the guide while replicating all steps on your machine.***

## How libraries are organized
It's nice to know how libraries are organized in clojure, because this will help you understand your namespace definitions and how to navigate inside documentations.<br/>

Most libraries are just a set of functions defined inside files that are in it's project folder structure.<br/>

During this guide, i'll put links to libraries API docs, and remember that they are just functions defined inside files on the github project folders, this helps claryfing how we actually visualize how we are importing functions to our namespace.

## Creating the project
Open your terminal and create a new clojure project using Leiningen:
```
lein new app
```

Enter in the app folder, here is the project structure:
```
.
├── CHANGELOG.md
├── LICENSE
├── README.md
├── doc
│   └── intro.md
├── project.clj
├── resources
├── src
│   └── app
│       └── core.clj
└── test
    └── app
        └── core_test.clj
```

## Import the library for HTTP
The first file we are going to edit is *project.clj*, this file is used by Leiningen to manage our project, and we can import libraries using it.<br/>
We are going to use the [Ring](https://github.com/ring-clojure/ring) library, Ring acts as an API for web servers.<br/>

Inside your *project.clj* edit the **:dependencies** key to the following:

```
:dependencies [[org.clojure/clojure "1.10.0"] 
               [ring "1.7.1"]] 
```

You can always download the dependencies you add to your project opening the terminal and running the command: 

```
lein deps
```

As we move on in this guide, you should take a look on the [Ring github page](https://github.com/ring-clojure/ring). If something is unclear or you get stucked, check their [wiki page](https://github.com/ring-clojure/ring/wiki) or it's [API docs](http://ring-clojure.github.io/ring/) to refer to what the functions does and expects as parameters.

## Let's fire up the server
Remember that the Ring library talks with a web server under the hood, in our case, the web server that we will use is  [Jetty](https://www.eclipse.org/jetty/), because ring already has an adapter for jetty written and ready to use for us.

Before we start the server, we need to write how this will happen.<br/>
Open the core.clj file at: **src/app/core.clj**

replace everything with this:
```
(ns app.core
  (:require [ring.adapter.jetty :refer [run-jetty]]))

(defn handler [request]
  {:status 200
   :headers {"Content-Type" "text/html"}
   :body "Hello world"})

(defn -main
  []
  (run-jetty handler {:port 3000}))

```

Allow me to explain what's happening here.<br/>

It's not enough to just import the libraries in our ***project.clj*** file. We need to explicitly call what we want from them inside our own project, in this case, our own namespaces.<br/>
Right after our namespace definition, we import the ***ring.adapter.jetty*** namespace functions, more specifically, the ***run-jetty*** function.<br/>

After this piece of code, we are defining what we call a ***Handler***.<br/>
The handler is nothing more than a function, which should accept an argument, which will be the HTTP request from the client, and should return a clojure hash-map, that contains basic info on how to respond this request, this way our adapter will interpret it and give the response to the client.<br/>
Basically:
```
client request > handler function 'handles it' > returns hash-map describing the http response
```
The hash-map response key-values is self explanatory if you examine it close enough.<br/>

Finally, we are defining a ***main*** function, which will be executed when we run our code. This function simply executes the [run-jetty function](http://ring-clojure.github.io/ring/ring.adapter.jetty.html) which is the adapter for the jetty web server, using our handler as a parameter and specifies a port. As the ring documentation says:<br/>

> Adapters are used to convert Ring handlers into running web servers.<br/>

Alright. Before we finally turn the server on, we need to specify to Leinigen (which we will use to run our code) that we do have a main function that should run by default. <br/>
Open the ***project.clj*** file again, and add this right before the enclosing parenthesis: 
```
:main app.core
```
This is telling leiningen as i said, that there is a main function, on the namespace app.core<br/>

Remember, if you get stucked, check how the code is written in the [Github repo](https://github.com/matthewlisp/clj-api-template) of this guide.<br/>

Let's try to run the server now, open the terminal inside your project folder and run the command:
```
lein run
```
If everything went smooth, open up your browser and go to: [localhost:3000](localhost:3000)<br/>
You should see the "Hello world" message which was the body of our response to any given request.

## Understanding how the HTTP request is represented
Ok, before we proceed, it's important to know how the HTTP requests are represented in the Ring library and more specifically in our application.<br/>

The easiest way to do this is by looking at the request, to do that, we're going to use a clojure function that prints data to the terminal in a more readable way, let's update our namespace require definitions:

```
(ns app.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [clojure.pprint :refer [pprint]]))
```

And now, let's update our handler function, because this is the function that receives our HTTP request:

```
(defn handler [request]
  (clojure.pprint/pprint request) ; Prints the request on the console 
  {:status 200
   :headers {"Content-Type" "text/html"}
   :body "Hello world"})
```

Remember that what our functions returns is the last expression, and that means we can execute other tasks before returning the result. Clojure is not a pure functional language, side-effects have its utility, when you are doing software for real use, such as debugging.

Now, while the server is running, you can look at the terminal window and see how the HTTP request is represented, let's take a look:

```
{:ssl-client-cert nil,
 :protocol "HTTP/1.1",
 :remote-addr "127.0.0.1",
 :headers
 {"cache-control" "max-age=0",
  "accept"
 "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
  "upgrade-insecure-requests" "1",
  "connection" "keep-alive",
  "user-agent"
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.132 Safari/537.36",
  "host" "0.0.0.0:3000",
  "accept-encoding" "gzip, deflate",
  "accept-language" "en-US,en;q=0.9"},
 :server-port 3000,
 :content-length nil,
 :content-type nil,
 :character-encoding nil,
 :uri "/",
 :server-name "0.0.0.0",
 :query-string nil,
 :body
 #object[org.eclipse.jetty.server.HttpInputOverHTTP 0x7b2512bb "HttpInputOverHTTP@7b2512bb[c=0,q=0,[0]=null,s=STREAM]"],
 :scheme :http,
 :request-method :get}
```

It's a simple Clojure map, we're going to work with it later, and leaving it here is perfect for debugging.

## Import the routing library

We have a very known routing library in clojure, it's specially made for Ring. This library is [Compojure](https://github.com/weavejester/compojure).

To start using it, let's update our ***project.clj*** file again, at the ***:dependencies*** key of the map, add:

```
[compojure "1.6.1"]
```
  
I recommend that later you take a deeper look at it's documentation.

## Creating routes

Before we start creating routes, we have to update again our namespace require definitions, take a look:

```
(ns app.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [clojure.pprint     :refer [pprint]]
            [compojure.core     :refer [routes GET]]
            [compojure.route    :refer [not-found]]))
```

What is happening here? I've added two functions from compojure.core and one from compojure.route, we're going to see them in action now as i create our http routes:

```
(def my-routes 
  (routes
   (GET  "/endpoint-a"  [] "<h1>Hello endpoint A</h1>")
   (GET  "/endpoint-b"  [] "<h1>Hello endpoint B</h1>")
   (not-found "<h1>Page not found</h1>")))
```

Here's what the code does:<br/>
Creates ***my-routes*** which is just a name reference to what our ***routes*** function returns.<br/>
The ***routes*** function, as the compojure [API documentation](http://weavejester.github.io/compojure/compojure.core.html#var-routes) says:<br/>

> Create a Ring handler by combining several handlers into one.<br/>

So, basically, ***my-routes*** is just a ring handler! we can pass 'several handlers' as arguments to ***routes*** function, and compojure have some macros ready for this, one of them is ***GET***, as you can see, the syntax is self explanatory, just keep in mind that what those macros returns under the hood is actually a response map, to proof this we are going to make something, bear with me, open up a REPL and do these steps:

1. Be sure your REPL is in the ***app.core*** namespace
2. If ***my-routes*** return a Ring handler, then it's a function, let's see


```
app.core> my-routes
#function[compojure.core/routes/fn--2512]
```

3. Yup! it's a function, and it's a Ring handler, so we can pass an HTTP request map? sure we can, let's use the one i paste here before:

```
app.core> 
(my-routes                          ; I've ommited the header info to shorten the code
 {:ssl-client-cert nil,
  :protocol "HTTP/1.1",
  :remote-addr "127.0.0.1",
  :headers {...} ,
  :server-port 3000,
  :content-length nil,
  :content-type nil,
  :character-encoding nil,
  :uri "/endpoint-a",              ; Notice i have changed the uri requested to /endpoint-a
  :server-name "0.0.0.0",
  :query-string nil,
  :body nil,
  :scheme :http,
  :request-method :get})
  
app.core> {:status 200, :headers {"Content-Type" "text/html; charset=utf-8"}, :body "<h1>Hello endpoint A</h1>"} 
```

There you go, the answer is a response map, just as expected, compojure is smart enough to create the response map automatically to us! 

The last part of our routes, is the ***not-found*** function, again let's look at what the compojure documentation says:<br/>

> Returns a route that always returns a 404 “Not Found” response with the supplied response body. <br/>

Well, this says pretty much everything, and why we need this function? It's because our routes will be tried one by one when the HTTP request comes, and if it doesn't match anything, it will raise an error and crash the application, but if we have this ***not-found*** function, it will return this anyway in the end if nothing matches.

Alright, the last thing we have to do is replace the handler used by ***run-jetty*** in our ***main*** function, simply because ***my-routes*** returns a handler and this handler takes care now of routing and that's what we wanted in this section, in the end our code is looking like this now: 

```
(ns app.core
  (:require [ring.adapter.jetty :refer [run-jetty]]
            [clojure.pprint     :refer [pprint]]
            [compojure.core     :refer [routes GET]]
            [compojure.route    :refer [not-found]]))
            
(def my-routes 
  (routes
   (GET  "/endpoint-a"  [] "<h1>Hello endpoint A</h1>")
   (GET  "/endpoint-b"  [] "<h1>Hello endpoint B</h1>")
   (not-found "<h1>Page not found</h1>")))

(defn -main
  []
  (run-jetty my-routes {:port 3000})) 
```

## JSON Requests and responses

We are setting up an API, so we need a data format to communicate with it, and by means of popularity i'll choose JSON to demonstrate here, and probably you're already thinking about how we are going to parse the requests and responses into json, the Ring+Compojure way to do this is to represent JSON as clojure hash-maps. To do the parsing from clojure maps to JSON, we use ready-to-go functions, that we call Middlewares.<br/>

This rises something important, we have to understand what a Middleware function is, in our context, middlewares are functions that add's some functionality (in other words, tweak the data) from handlers, so they receive a handler, mess with the handler and returns it again but with the new functionality added, heres the basic flow:

```
(middleware-fn handler) -> tweaked-new-handler
```

ForJSON the two critical middlewares comes from the [Ring-JSON](https://github.com/ring-clojure/ring-json) library, they are:<br/>
* ***wrap-json-response***<br/>
* ***wrap-json-body***<br/>

But as always, before we use it, we have to update our ***project.clj*** again by importing this lib in our project, open the file and at the ***:dependencies*** key of the map, add:

```
[ring/ring-json "0.5.0"]
```

Cool. Because this iteration on our code makes many modifications, i'll paste it here and we are going to breakdown the changes, here we go:

```
(ns app.core
  (:require [ring.adapter.jetty   :refer [run-jetty]]
            [clojure.pprint       :refer [pprint]]
            [compojure.core       :refer [routes GET POST]]
            [compojure.route      :refer [not-found]]
            [ring.middleware.json :refer [wrap-json-response wrap-json-body]]
            [ring.util.response   :refer [response]]))

(def my-routes
  (routes 
   (GET  "/endpoint-a"  []      (response {:foo "bar"}))
   (GET  "/endpoint-b"  []      (response {:baz "qux"}))
   (POST "/debug"       request (response (with-out-str (clojure.pprint/pprint request))))
   (not-found {:error "Not found"})))

(def app
  (-> my-routes
      wrap-json-body
      wrap-json-response))

(defn -main
  []
  (run-jetty app {:port 3000}))
```

At the namespace definitions, i've just added the ***ring.middleware.json*** from the Ring-JSON library and the two critical functions. Also now we have ***ring.util.response*** refering the ***response*** function, and you'll find out what it is in a sec.<br/>

P.S: i've also added the ***POST*** macro from ***compojure.core***!<br/>

Before we discuss the changes in our routes, let's see this new definition that i created called ***app***.<br/>

What is this doing? this is **wrapping middlewares** to our ***my-routes*** handler. And i'm using the threading macro ***->*** because otherwise this code would start being ugly to read, as we possibly will need more middlewares in the future, and things would start being like this:

```
(wrap-blablabla (wrap-json-bdoy (wrap-json-response my-routes)))
```

*(((((())))))* yeah that's what i'm talking about, so using threading macros are useful for avoiding readability issues.

So in the end, ***app*** is just our good old handler from the beginning. That's precisely why we are now using it as the first argument for the ***run-jetty*** function!!

Important to know now, is what the ***response*** function that i've refered does and why is it here. From the Ring docs:<br/>

> Returns a skeletal Ring response with the given body, status of 200, and no
headers.<br/>

Unfortunately, compojure can't handle clojure hash-maps treated as JSON and just pass this inside a response map as it did before for us, so we need this function.

Ok, now the fun part, our routes. First let me talk about our */debug* which is a ***POST*** route. I've created this route for debugging purposes, so we can see how our HTTP request looks like inside clojure. To do that i've used the ***pprint*** function that i mentioned earlier, but it needs some tweakes to print on the browser/terminal whatever client you're using to access the endpoint, that's the why of the ***with-out-str*** function. Note also, we are passing around a value that i called *request*, the reason is explained at compojure wiki [here](https://github.com/weavejester/compojure/wiki/Destructuring-Syntax#regular-clojure-destructuring).

The rest is pretty much self explanatory, i've used the ***response*** function i mentioned to create the response map, and passing the body to the ***response*** function. The body being a clojure map, is treated as a JSON, because the ***wrap-json-response*** middleware is handling this for us, as well as ***wrap-json-body*** is handling JSON coming from HTTP requests and transforming them into clojure maps, let's see all this in action:<br/>

* Update your core.clj using the code above
* Open the terminal on the project folder and do: lein run
* Open another terminal window and usgin the curl tool, we are going to check the debug endpoint:

```
$ curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST http://localhost:3000/debug
```

The output:<br/>

```
{:ssl-client-cert nil,
 :protocol "HTTP/1.1",
 :remote-addr "127.0.0.1",
 :params {},
 :route-params {},
 :headers
 {"user-agent" "curl/7.58.0",
  "host" "localhost:3000",
  "accept" "*/*",
  "content-length" "34",
  "content-type" "application/json"},
 :server-port 3000,
 :content-length 34,
 :compojure/route [:post "/debug"],
 :content-type "application/json",
 :character-encoding "UTF-8",
 :uri "/debug",
 :server-name "localhost",
 :query-string nil,
 :body {"key1" "value1", "key2" "value2"},
 :scheme :http,
 :request-method :post}
``` 


As you can see, now our ***:body*** key has a value of our JSON encoded request, and we are receiving as clojure hash-map, theres one thing to notice though, the keys are in *string* format, not in *symbols*, if you want to receive them as *symbols* (it simplifies the process of taking values from the request) you should read the [documentation of Ring-JSON](https://github.com/ring-clojure/ring-json) library, there is a middleware for doing this!

Let's also test one of our endpoints, the */endpoint-a*:

```
$ curl http://localhost:3000/endpoint-a
```
The output:<br/>
```
{"foo":"bar"}
```

As expected, and if you open your browser, you're going to see that you are indeed getting a JSON response.

## What we accomplished

Although we are lacking lot's of stuff for an API logic, you already have what i promissed in the beginning, which was creating a very simple code set-up to start developing yourself. Don't worry, i'll link you some resources to keep going on, and also will create part II of this post!

## A few tips & advices

I have some tips for you before going ahead:<br/>

* Clojure projects have a "connect the pieces" kind of way of building things, if you need something, there will be a library! Here's a gift: [Clojure ToolBox](https://www.clojure-toolbox.com/)
* As i said before, there's no correct way to build your system/project/API as long as your code is idiomatic and readable, you can't go wrong if you plan ahead and analyse your needs. [Clojure style guide](https://github.com/bbatsov/clojure-style-guide)
* Remember, you'll get stucked at some point ALWAYS because we are using an approach where you reuse pre-built pieces of your system, you NEED to read library documentations, they are always on their github repo and clojure libs docs never disappointed me.
* When everything seems to fail, search for answers on Clojure communities, if there's no answer then ask, you can google those communities but here is the most active (i think): [Clojure Slack](clojurians.net)

## What's next

The next logical step would be Validating input from the client, i'll cover this on another post using this project, but if you want to go by yourself, to validate input in clojure we use mostly [Specs](https://clojure.org/guides/spec), and i've already used [this](https://github.com/plumatic/schema) library too.<br/>

For tests you can start with [this](https://clojure.github.io/clojure/clojure.test-api.html) overview.<br/>

An even further step is using databases, here's the [entry point](https://github.com/seancorfield/next-jdbc).<br/>

Thanks for reading! If you need help, reach me on [Telegram](https://t.me/crytek) or [Twitter](https://twitter.com/MatthewLisp).