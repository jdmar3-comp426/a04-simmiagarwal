[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-f059dc9a6f8d3a56e377f745f24479a46679e63a5d9fe6f495e02850cd0d8118.svg)](https://classroom.github.com/online_ide?assignment_repo_id=6409265&assignment_repo_type=AssignmentRepo)

# a04 Build an API from scratch

In this assignment, we will build, from scratch, an API that talks to a database of users. Eventually, we will use this method (and this exact database) to add authentication to a page that you previously created in a01. This is all a lot easier than it seems at first.

There are a few basic concepts germane to this assignment that you will want to familiarize yourself with:

- REST (REpresentational State Transfer)
- CRUD (Create / Read / Update / Delete)
- SQL (Structured Query Language)

These concepts are explained in some detail in mod04, so you’ll want to have a look at that while we are working.

We are also going to use Express.js, which is a very simple webserver, to get us going without too much setup.

You’re starting with a package.json file that has no dependencies installed. We will install them here.

## Get started

First things first, let’s install the dependencies we need to get going. We’ll start by installing Express.js, which we will use as a web server for our API.

```npm install express```

Then we need to install the `md5` package. This is a package that allows for the hashing of strings using the MD5 algorithm. We will use it for passwords.

```npm install md5```

Finally, we need a database. We are going to use a JS implementation of a sqlite3 database. For this, we need to install `better-sqlite`, which is a less vulnerable implementation of sqlite3 in JS than some of the other similar Node packages available.

```npm install better-sqlite3```

Now that our dependencies are installed, we should look at our file structure.

You will have only two main files for this assignment, `server.js` which contains the bulk of our code to create the API and `database.js` which deals with the database. Skeletons of these files have been created for you. You will fill in the function definitions based on the instructions below.

> You will notice that the examples below are using a slightly different method for defining functions. This is intentional, because I want you to be able to recognize both. What you will see here is an example of ES6 “arrow” functions. There are pros and cons to using this method, which we will discuss. The major things to be aware of is that they are:
1. not self-referencing, which means that they cannot be invoked from within themselves; and they are
2. not named. They are anonymous, which means that they can be a little harder to debug. They are much simpler, syntactically.

## Set up a server

Let’s look in our files.

Your `server.js` file needs some additions to get you started. Open it up and find the lines near the top where we have some variable definitions. It should look like this:

```
// Define app using express
var express = require("express")
var app = express()
// Require database SCRIPT file

// Require md5 MODULE

// Require a middleware extension for express 
var bodyParser = require("body-parser");
```

Here we have a few definitions and we need to add two more in order to make this script work with our other `database.js` script. Define, where the comments indicate, the following:

1. a `var` called `db` to require the `database.js` script, and
2. a `var` called `md5` to require `md5` module.

There are already other modules being called by require so you can use those as examples to infer how to do the above. Since you’re requiring `database.js` as a file, you need to use a relative path, unlike with other node modules (e.g. `"./database.js"`).

Then find a comment that refers to a server port and define `HTTP_PORT` as 5000.

Once you have done that, `server.js` will allow you to run the server and check that it works. `start` is defined as a script in `package.json` to allow us to easily run our server.

This is what you should see if you run `npm run start`:

```
$ npm run start

> a04@1.0.0 start
> node server.js

Server running on port 5000
```

You should also be able to go http://localhost:5000 and see a message from the server. Find where this message is defined in server.js and change it to display “Your API is working!” when you go to the above address.

This means that your API is up and running! It doesn’t have any endpoints other than the root endpoint and a catchall that will return a 404 error, but that is enough for now. We will come back to this again after we look at what our database does.

## Set up a database

We are using the better-sqlite3 module to both set up our database and also define the way that our new API will interact with the database. Keep the documentation for better-sqlite3 handy. It will be useful to you and necessary.

Your starter code has a database set up already for you. When you run `server.js` after making the edits above, it will load `database.js` and check to see if there is a database named `users.db`. If there is not, then it will create one and populate it with two users. You should be able to identify in the code provided where this is happening.

Let’s start with those users and see what else we can do with the database.

## Database interactions

In this assignment, we’re using SQLite3, which is a standalone instantiation of a SQL database. If we used something like MySQL, then we would need to set up a database server, etc. This allows us to work with a database without having to build a bunch of infrastructure for ourselves.

For this reason, SQLite3 is a good tool for learning and teaching. You will likely not use it in practice, however, and instead use an actual MySQL or other database server. For the purposes of this and other assignments requiring the use of a database, use SQLite3.

Most SQL features are available in SQLite3, with some minor differences. Look at the official SQLite site for a rundown of how SQLite understands SQL.

There are one million resources available online for looking up and understanding SQL statements. Here is a good SQL cheat sheet from SQL Tutorial. Start there and Google away for others.

We need to look at a few things in our database before we return to our API. Let’s start with those users already in our database and see what else we can do with the database.

## SQL statements

The function of our API will be to Create, Read, Update, and Delete (CRUD) records in our database. To do this, we need to define some SQL statements that do the following:

1. Create a new user with password in the database;
2. Read all user information in the database (this one is already defined for you);
3. Read a single user’s information in the database;
4. Update a single user’s information in the database; and
5. Delete a single user from the database.

We will first hammer out what these statements need to look like for the above specific cases. Then we will wrap them in functions from better-sqlite so we can use them to define our API endpoints later.

## Read some records with SELECT

Start with the second member of the list above. We need to get all the records in our database. To get records out of a SQL database, we use `SELECT`.

SQL syntax should be pretty parseable. That is, if you just read a SQL statement out loud to yourself, it should make some sense to you.

Try it with this:

```
SELECT * FROM userinfo
```

What is happening here?

We are selecting ALL ("*" means ALL) records from a table called “userinfo”. This should return every record held in the table created by `database.js`.

Where do we put this, though?

If you look in `server.js`, you should be able to find the above statement wrapped by better-sqlite functions as part of the definition for API endpoint.

```
// READ a list of users (HTTP method GET) at endpoint /app/users/
app.get("/app/users", (req, res) => {   
        const stmt = db.prepare("SELECT * FROM userinfo").all();
        res.status(200).json(stmt);
});
```

Let’s break this apart by looking at lines 3-4. `prepare()` is what we use to set up the SQL statement. `all()` returns every record responsive to the query in the statement, which in this case should be every record. Here `all()` is appended to `prepare()`, but you could also write this database call as follows:

```
const stmt = db.prepare("SELECT * FROM userinfo");
const all = stmt.all();
```

We will look at what the rest of the function is doing in the next section.

For now, we want to do something similar this with our 4 other SQL statement prompts.

```
SELECT * FROM userinfo WHERE id = 2
```

This is effectively the same statement, but with a conditional that looks for a specific `id`.

Look at the better-sqlite statements class under `all()` and `get()` to find the relevant functions for the purposes of wrapping these two statements. `get()` does something similar to `all()` but instead of returning every row, it returns one row.

## Create a new record with INSERT

Here is a SQL statement that creates a new record with a username and password.

```
INSERT INTO userinfo (user, pass) VALUES (?, ?)
```

For INSERT, UPDATE, and DELETE statements, look at the better-sqlite statements class under `run()`. Pay special attention to the `info` object that `run()` returns. It is useful for generating messages in your API.

## Update a record with UPDATE

Here is a SQL statement that can update an existing record matching a specific id with either a new username, new password, or both. The COALESCE verb allows a value to be updated if a new one is introduced and remain the same if there is no new value.

```
UPDATE userinfo SET user = COALESCE(?,user), pass = COALESCE(?,pass) WHERE id = ?
```

## Delete a record with DELETE

Here is a SQL statement that deletes a record matching a specific id.

```
DELETE FROM userinfo WHERE id = ?
```

## Translating between languages

The following table translates verbs from our CRUD conceptualization to SQL statements and HTTP method.

|CRUD 	|SQL 	|HTTP 	|What does it do?|
| -- | -- | -- | ------|
|Create 	|INSERT 	|POST 	|Makes a new record.|
|Read 	|SELECT 	|GET 	|Retrieves an existing record.|
|Update (replace) 	|REPLACE 	|PUT 	|Searches for a record and overwrites it. If it doesn’t exist, the record is created.|
|Update (modify) 	|UPDATE 	|PATCH 	|Modifies the information in an existing record.|
|Delete 	|DELETE 	|DELETE 	|Removes an existing record.|

Some of the verbs are identical. Others, not so much.

This will be useful as we create endpoints.

## API endpoints

For this assignment, you need to define 5 endpoints for your API (one has already been defined for you to return all records). Below are descriptions of each endpoint and a test to check if they are working.

I have set the assignment up using Express.js, which simplifies the createion of endpoints and various other tasks by providing a library of functions tailored to these tasks. There are other similar libraries and frameworks. This is one way of doing what we are aiming at doing.

Refer to the Express documentation as you work through createing the endpoints.

## Response

Each endpoint should perform the intended function and then return a status code and a message as a JSON object. That will look something like this using Express:

```
res.status(200).json({"message":"OK (200)"})
```

The first part of the above snippet, `res.status()`, is a status code response. These will not all be the same. Refer to this status code documentation to identify the appropriate status code. There is a also list of all available status codes.

The second part of the above snippet could be written as a standalone: `res.json()` as well. As written, it outputs a JSON message that can then be used by a client interface.

## Request

The requests for the API endpoints will have variables in Express. Some of these are included in the URL that represents the endpoint itself and others can be passed either as JSON or as a query appended to the URL.

Variables that you include as part of the path in the endpoint become parameters. So, if you are using `id` in the endpoint path, then you use `req.params.id`.

For example: the `id` part of `/app/user/:id` will be represented as `req.params.id`.

Variables that you include as part of a URL-encoded query are part of the body content that gets handled by a piece of middleware called a body parser. If you are using `user` and `pass` in a query string, then you use `req.body.user` and `req.body.pass`, respectively.

For example: if you have a URL like `/app/update/user/:id/?user=test&pass=supersecurepassword` then user will be represented as `req.body.user`.

## Passwords

We’re not doing really anything with security in this assignment, but it is a good idea to think about what you are passing as cleartext. Passwords, as a rule, are never going to be stored in cleartext.

For our purposes, we’re going to use `md5()` to wrap password so that we will have this: `md5(req.body.pass)`. For the example password above, instead of “supersecurepassword” in the database, this will put something like “20f98f38b3bbd2da93d75fd25806c048.”

## Express endpoints

Express has functions that correspond to the HTTP methods listed in the table above.

The table below has a list of the endpoints you will need to define and the Express functions to use.
|Endpoint 	|Express function 	|What should it do?|
|--|--|--|
|/app/new/user 	|app.post() 	|Create a new record in the database with a username and password.|
|/app/users 	|app.get() 	|Retrieve all existing records.|
|/app/user/:id 	|app.get() 	|Retrieve an existing record with the specified ID.|
|/app/update/user/:id 	|app.patch() 	|Update the username and/or password for an existing record with the specified ID.|
|/app/delete/user/:id 	|app.delete() 	|Delete an existing record with the specified ID.|

## Endpoint tests

These are the tests that we will run against your API to autograde a04. Below each command is the expected STDOUT

```
message":"1 record deleted: ID 2 (200)"}
$ curl -X GET http://localhost:5000/app/
{"message":"Your API works! (200)"}
$ curl -X GET http://localhost:5000/app/users/
[{"id":1,"user":"admin","pass":"bdc87b9c894da5168059e00ebffb9077"},{"id":3,"user":"jdmar3","pass":"553d1b0339e9565f8e11922ad16fbf3f"}
$ curl -X GET http://localhost:5000/app/user/2/
{"id":2,"user":"test","pass":"9241818c20435c6672dac2c4b6e6c071"}
$ curl -X POST -d "user=newtest&pass=supersecurepassword" http://localhost:5000/app/new/
{"message":"1 record createed: ID 3 (201)"}
$ curl -X PATCH -d "user=oldtest&pass=adifferentpassword" http://localhost:5000/app/update/user/2/
{"message":"1 record updated: ID 2 (200)"}
$ curl -X DELETE http://localhost:5000/app/delete/user/2/
{"message":"1 record deleted: ID 2 (200)"}
```