---
layout: post
title: "Building a backend for backbone.js Todos example with Grails and MongoDB"
date: 2012-01-31 18:09
comments: true
published: true
categories: 
- Grails
- backbone.js
- MongoDB
- JavaScript
---

In case you haven't yet tried it yet, [Backbone.js](http://documentcloud.github.com/backbone/) is a wonderful little JavaScript MVC framework. Ditto for [Grails](http://grails.org/)! And soon you'll see that they are a great match for each other - in this post, we'll build a backend for the Backbone.js ToDo example using Grails and MongoDB.

You can fork the completed example application [here](https://github.com/lauripiispanen/grails-backbone-mongodb-example).

<!-- more -->

First, let's create the Grails application and install mongodb plugin. We'll also uninstall the default hibernate plugin. I'll be using the new Grails 2.0 interactive console throughout this entry. First, edit _grails-app/conf/BuildConfig.groovy_ and replace this line:

	runtime ":hibernate:$grailsVersion"

with this to install the Grails MongoDB plugin:

	runtime ":mongodb:latest.integration"

...and configure Grails to mount into root context in development mode. Add this to _grails-app/conf/Config.groovy_:
	
	grails.app.context = "/"

and run the app with _run-app_:

	$ grails
	grails> run-app
	
Next up, let's copy the .js, .css and .png files from the standard backbone.js todo example into our _/web-app/_ folder, and overwrite our _grails-app/views/index.gsp_ with _index.html_ from the example app. At this point the index file will appear if you start the app, but none of the JavaScript code is working - it's pointing to _/test/vendor/_ etc. We'll change the app to use the Grails 2.0 Static Resources plugin and its bundles. You should have the following files in place:

	web-app/backbone.js
	web-app/underscore.js
	web-app/todos.js
	web-app/destroy.png
	web-app/todos.css

First, we create the resources bundle for our app in _grails-app/conf/TodoResources.groovy_:

	modules = {
	    todo {
	        dependsOn 'jquery, underscore, backbone'
	
	        resource url: '/todos.css'
	        resource url: '/todos.js'
	    }
	    backbone {
	        resource url: '/backbone.js'
	    }
	    underscore {
	        resource url: '/underscore.js'
	    }
	}

And then we create a layout to include our bundle on the page. You can repurpose the existing _grails-app/views/layouts/main.gsp_ to suit our needs.

	<!doctype html>
	<html>
	        <head>
	                <title><g:layoutTitle default="Grails"/></title>
	                <g:layoutHead/>
	                <r:layoutResources />
	        </head>
	        <body>
	                <g:layoutBody/>
	                <r:layoutResources />
	        </body>
	</html>

And then we take the layout in use by removing all css links from _index.gsp_ and replacing them with:

	<r:require module="todo" />
	<meta name="layout" content="main">

At this point your todo application should _almost_ work. It's complaining about the missing localstorage plugin, which is fine because we're about to replace it anyway. So just remove this line from _todos.js_:

	localStorage: new Store("todos"),

and replace it with:

	url: '/todos',

But now the todos app is complaining that it can't find that URL. Which is fine - that just means that its happily trying to get our list of todos from the server, but can't. No problemo, so that means we need to create a controller that will be our main AJAX endpoint.

	grails> create-controller todo.TodosController

...which nicely maps to the aforementioned _/todos_ thanks to Grails default conventions. Sweet! We'll also add an action to the controller that gives us a list of todo items. For now we'll just return an empty list in JSON. This is where Grails converters come really handy:

	package todo
	
	import grails.converters.JSON 
	
	class TodosController {
	
	    def index() {
	        render( [] as JSON )
	    }
	}
	
At this point our app is _nearly_ working! We can see Backbone POSTing changes to our todos to the server. But nothing's being persisted yet, so if we refresh the page, our todo list is blank again. So, what we need in order to persist our data, is a domain class that mirrors the one that we have in our Todo backbone.js example:

	grails> create-domain-class todo.Todo

In _grails-app/domain/todo/Todo.groovy_:

	package todo
	
	class Todo {
	
	    boolean done
	    int order
	    String text
	
	    static constraints = {
	        text(nullable: false, empty: false)
	    }
	}

And we'll also change our controller action to list all of our todo items:

	def index() {
	    render( Todo.findAll() as JSON )
	}

But so far we don't have any todos in the database, so the list is empty! Looking at AJAX traffic in Web Inspector, we see that whenever todos are updated, Backbone syncs them with the server via POSTs.

{% img /images/posts/web_inspector.png %}

So, to create some nice RESTful URLs for our todo app, we'll just define them in _grails-app/conf/UrlMappings.groovy_:

	"/todos/$id?"(controller: "todos") {
	    action = [GET:"list", POST: "save", DELETE: "delete", PUT: "edit"]
	}
	
And a corresponding action to bind incoming POST data to a new todo item and save it:

	def save() {
	    def todo = new Todo(request.JSON)
	    render( todo.save() as JSON )
	}

BAM! Our new todo items are persisted in MongoDB! Just like you guessed, DELETE is just as simple.

	def delete() {
		def todo = Todo.findById(params.id)
		todo?.delete()
		render(todo as JSON )
	}
	
Now we can add and remove items. This is all nice and cool. However, what would a todo app be without a chance to complete some items? Backbone calls _.save()_ on each update of a model item, and that translates to a PUT request to our backend. Based on our URL mappings, we'll still need to implement an _edit()_ action:

	def edit() {
		def todo = Todo.findById(params.id)
	    bindData(todo, request.JSON)
	    render(todo.save() as JSON )				
	}

And that's it! Our todo state now successfully propagates to the server! Job well done... oh wait! When we click on the checkboxes, nothing happens. And the original todo demo has a cool _"X tasks left"_ that is not displayed in our application. What gives? Let's take a look at the Grails view template we use - bring up _index.gsp_:

	<% if (total) { %>
	    <span class="todo-count">
	        <span class="number"><%= remaining %></span>
	        <span class="word"><%= remaining == 1 ? 'item' : 'items' %></span> left.
	    </span>
	<% } %>
	<% if (done) { %>
	    <span class="todo-clear">
	        <a href="#">
	            Clear <span class="number-done"><%= done %></span>
	            completed <span class="word-done"><%= done == 1 ? 'item' : 'items' %></span>
	        </a>
	    </span>
	<% } %>

A-ha! There's some underscore templates. But those look just like GSP scriptlets! And just as well, they don't show up in the generated markup. We need to fix those. Luckily there is a way to change how underscore interpolates the templates. Just put this at the top of _todos.js_:

	_.templateSettings = {
	    interpolate : /\{\{(.+?)\}\}/g,
	    evaluate : /\{!(.+?)!\}/g
	};
	
And now we can change our templates to a nicer (nicer to .gsp that is) format. Just change all _<%=_ and _%>_ scriptlets to _\{\{_ and _}}_ respectively, and _<%_ and _%>_ to _{!_ and _!}_ and voila! A complete todo application! Albeit there's no authentication and multi-user and and and... Perhaps we'll add those later!