---
layout: post
title: Collect Email Addresses with Node and Postmark
---

[Sample Application for Node Email Collector is Here GitHub.](https://github.com/drquinn/nodememails)

How hard could it be to collect email addresses from interested people on your website? Well, not very hard with Node, Express, MongoDB, and Postmark.

#Overview

The goal is to allow people to be notified when our startup is further along and actually has a product for sale. How do we gather those email addresses? Where are they stored? How do we send a confirmation email to let the user know they were added to the list?  All of these questions and more will be answered by mixing a few technologies.

#Build The Base Structure Of The Application

###Set Up Node And Express

Node will be running our application. This allows us to write JavaScript which Google's V8 Engine will convert to machine code to execute with extremely high performance.
Visit the [Node.js](http://nodejs.org) website and download the binaries if you do not already have it installed.

Create a folder for the project to live in. Mine is called **nodememails**.

	mkdir nodememails
	cd nodememails
	npm init

After creating the folder with **mkdir**, move into that folder, then run **npm init**. This will initialize a package.json file for you and walk you through filling it out.  Once you have your package.json, update it with the following dependencies:

	{
	  "name": "nodememails",
	  "version": "0.0.0",
	  "private": true,
	  "scripts": {
	    "start": "node ./app.js"
	  },
	  "dependencies": {
	    "body-parser": "~1.10.2",
	    "express": "~4.11.1",
	    "mongodb": "*",
	    "monk": "*"
	  }
	}


Express is a framework that runs on top of Node to make it simple to gain web server functionality. We could set up our routes and API using Node alone with its HTTP class, but express makes it even quicker to set up and easy to manage.

Install the dependencies.

	npm install

This will install everything in our package.json under dependencies: Body-Parser helps us retrieve data from an HTTP POST, Express, MongoDB  is where our email addresses will be stored, and Monk will allow us to send commands to MongoDB.

In your root directory (nodememails for me), add an app.js file which will run our express routes and much of the server code.

Also, add a folder called data. This is where we store our mongodb database.

	mkdir data

###Install MongoDB
I'm running on a Linux Mint box, but you can find instructions for whatever operating system you run on at [MongoDB's website](http://docs.mongodb.org/).  Here are the steps I took.

	sudo apt-get install mongodb-org

	mongod --dbpath pathtodata

Where *pathtodata* is the full path to your data folder.  You now have MongoDB running with its database stored in that data folder. If you don't add the --dbpath parameter, MongoDB will store your database at /data/db/. 

###Set Up The Database

Now we will work with the MongoDB shell to interact with the database and set a few things up.  Open the Shell.

	mongo

If MongoDB is set up correctly and running, you will see the shell connect to your database.

	mongo
	MongoDB shell version: 2.4.9
	connecting to: test

If you get an error here, move back and ensure you have MongoDB set up properly.  Now in the shell, let's make a database called **emails** and collection called **usercollection** to store email addresses along with a user's name.

	use emails

This does not actually create the database yet. To create the database, we simply add a JSON object to a collection within our database. This ability to work directly with JSON objects will make communication with Node and JavaScript feel very natural.

	newdata = [{ "name" : "user1", "email" : "user1@email.com" }, 
			   { "name" : "user1", "email" : "testuser3@testdomain.com" }]

	db.usercollection.insert(newdata)

As you can see we are storing a name and email address for every user that enters information into the form.

###Create Basic HTML Form
Here we will create a bare bones HTML page for a user to enter their information.  In this example, I am creating a folder called **public** in my root directory. This is the folder arrangement used by the Express generator and one that I typically follow for web projects like this. 

In the **public** folder, create **collection.html**. Here is the basic HTML I am using.

	<html>
	<head>
	</head>
	<body>
	    <form id="mailForm" action="addMail" method="post">
	        <div>
	            <label for="name">Name: </label>
	            <input type="text" id="name" name="name"></input>
	            <br /><br />
	            <label for="email">Email: </label>
	            <input type=text" id="email" name="email"></input>
	        </div>
	        <br /><br />
	        <input type="submit" value="Submit"></input>
	    </form>
	</body>
	</html>

I am leaving this without style for simplicity. Bring in something like Bootstrap or Foundation to make this quickly look much better.  Next we will set up the routes in order to navigate to this HTML page.


#Build Routes

Now that we have the structure complete, we can set up a simple Express API. It will do the following:

* Present a form for users to enter their name and email address.
* Expose a POST request for adding the user to our database.
* After the database insert, call a function to send them an email using Postmark.

Here is what your **app.js** file (the one we created above in our project's root directory) should look like after adding routes.

	// Get needed dependencies.
	var express = require('express');
	var path = require('path');
	var bodyParser = require('body-parser');
	
	var mongo = require('mongodb');
	var db = require('monk')('localhost:27017/emails');
	var app = express();
	
	// bodyParser() gets the data from a POST.
	app.use(bodyParser.json());
	app.use(bodyParser.urlencoded({ extended: true }));
	
	// Middleware that injects the database object into each request.
	// This gives us easy access to update the database in other 
	// api calls.
	app.use(function(req,res,next) {
	    req.db = db;
	    next();
	});
	
	// Get an instance of the Express Router.
	var router = express.Router();
	
	// Get the email submission form.
	router.get('/collect', function (req, res) {
	    res.sendFile(path.join(__dirname, 'public', 'collection.html'));
	});
	
	router.post('/addMail', function(req, res) {
	    var username = req.body.name;
	    var useremail = req.body.email;
	   
	    db.get('usercollection').insert({
	      "name" : username,
	      "email": useremail
	    }, function (err, doc) {
	      if (err) {
	         res.send("Error adding user to db.");
	      }
	      else {
	        sendEmail(useremail);
	        res.location('/');
	        res.redirect('/');
	      }
	    }); 
	});
	
	// Tell express to use the route you just set up.
	app.use(router);
	
	app.listen(3000);
	console.log("listening on port: 3000");

#Bring in Postmark
This completes our route setup. The only thing missing is the Postmark setup. If you do not already have an account, create one at [postmarkapp.com](https://postmarkapp.com) 

We will be sending an email using the [Postmark REST API](http://developer.postmarkapp.com/developer-send-api.html). It is very easy to work with. Here is the missing sendEmail() function. You can add this to the app.js file just below the final console.log().

	var http = require('http');
	var apiToken = 'POSTMARK_API_TEST'
	function sendEmail (toEmail) {
	    
	    var emailbody = {
	       'From': "test@fromemail.com",
	       'To': toEmail, 
	       'Subject': 'Postmark test', 
	       'HtmlBody': '<html><body><strong>Hello</strong> dear Postmark user.</body></html>'
	    };
	
	    var options = {
	        host: 'api.postmarkapp.com',
	        path: '/email',
	        method: 'POST',
	        headers: {
	            'Accept': 'application/json',
	            'Content-Type': 'application/json',
	            'X-Postmark-Server-Token': apiToken
	        }
	    };
	
	    var req = http.request(options, function(res) {
	        res.setEncoding('utf8');
	        res.on('data', function (chunk) {
	            console.log('body: ' + chunk);
	        });
	    });
	
	    req.write(JSON.stringify(emailbody));
	    req.end();
	}

You will need to require Node's HTTP module to send the HTTP Request to Postmark. The options are configured as expected by Postmark. As you can see, the function **sendEmail** takes a string **toEmail** which we send from the **/addMail** POST route in our Express API.

###Conigure Postmark With Your Account Settings

The apiToken will run with the test value POSTMARK\_API_TEST. But you will need to replace that with your API Token to send through your Postmark account. 

Update the properties in emailbody with your settings as needed. The 'From' property must also match the send as email in your Postmark account.

'HtmlBody' should include the email you plan on sending out.


#Finished

You now have a basic email enabled form that can store names and addresses for you. Grab the sample application I created for this tutorial on GitHub:

[https://github.com/drquinn/nodememails](https://github.com/drquinn/nodememails)

Check out my implementation of this at [Manuvr](http://manuvr.io). Click on the link to add yourself to our mailing list if you have any interest in gesture control or using Node to handle large amounts of streaming data.

