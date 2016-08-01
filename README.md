# simple-express
a simple express following the article named &lt;THE DEAD-SIMPLE STEP-BY-STEP GUIDE FOR FRONT-END DEVELOPERS TO GETTING UP AND RUNNING WITH NODE.JS, EXPRESS, JADE, AND MONGODB>


### MongoDB

##### Create a database
use the command to create and switch a database:

    use simple-express

add a database name 'simple-express'.

##### Add some data
Now, I want to add some data like :

    {
        "_id" : 1234,
        "username" : "cwbuecheler",
        "email" : "cwbuecheler@nospam.com"
    }
    
You can create your own _id assignment if you really want, but I find it's best to let Mongo just do its thing. 
It will provide a unique identifier for every single top-level collection entry.
Use the following commands:
    
    db.usercollection.insert({ "username" : "testuser1", "email" : "testuser1@testdomain.com" })
    
Something important to note here: that `db` stands for our database, which as mentioned above we've defined as "simple-express".
The "usercollection" part is our collection. Note tha there wasn't a step where we create the "usercollection" collection. That's because the first time we add it, it's going to be auto-created.
If u want to see.type:

    db.usercollection.find().pretty()
    
the `.pretty()` method gives us linebreaks. It will return:

    {
    	"_id" : ObjectId("579e014f3652d01e57bbebde"),
    	"username" : "testuser1",
    	"email" : "testuser1@testdomain.com"
    }
    
Mongo is automatically generating ObjectID.
 
We can add a a couple more. Type:
 
    newstuff = [{ "username" : "testuser2", "email" : "testuser2@testdomain.com" }, { "username" : "testuser3", "email" : "testuser3@testdomain.com" }]
    db.usercollection.insert(newstuff);
    

### List in page

##### Expect

We want to generate the following HTML:

    <ul>
      <li><a href="mailto:testuser1@testdomain.com">testuser1</a></li>
      <li><a href="mailto:testuser2@testdomain.com">testuser2</a></li>
      <li><a href="mailto:testuser3@testdomain.com">testuser3</a></li>
    </ul>  

##### Connect to MongoDB
Connect with website and MongoDB instance.
Add three lines in app.js:

    var express = require('express');
    var path = require('path');
    var favicon = require('serve-favicon');
    var logger = require('morgan');
    var cookieParser = require('cookie-parser');
    var bodyParser = require('body-parser');
    
    // New Code
    var mongo = require('mongodb');
    var monk = require('monk');
    var db = monk('localhost:27017/simple-express');
    
Then use db before router take effect.
Find follow codes in app.js:

    app.use('/',roures);
    app.use('/users/, users);
    
Above the two lines just mentioned, add the following:

    //Make our db accessible to our router
    app.use(function(req, res, next){
        req.db = db;
        next();
    });
    
**If not put this above the rouing staff mentioned above (app.use('/', routes);), your app will not work.**

##### Display data list

Add router in `index.js`, and get `db` by `req`.
Basically, we tell our app which collection we want to use('userconlliction') and do a find, then return the result as the variable "docs".
Once we hace those documents, we then do a render of userlist ( which will need a corresponding Jade template), giving it the userlist variable to work with, and passing our database documents to that variable.

Then add userlist jade template.

    extends layout
    
    block content
        h1.
            User List
        ul
            each user, i in userlist
                li
                    a(href="mailto:#{user.email}")= user.username
           
When run `npm start`, see some errors:

    Error: Module version mismatch. Expected 48, got 44.
        at Error (native)
        at Object.Module._extensions..node (module.js:568:18)
        at Module.load (module.js:458:32)
        at tryModuleLoad (module.js:417:12)
        at Function.Module._load (module.js:409:3)
        at Module.require (module.js:468:17)
        at require (internal/module.js:20:19)
        at Object.<anonymous> (...../simple-express/node_modules/.npminstall/bson/0.2.22/bson/ext/index.js:15:10)
        at Module._compile (module.js:541:32)
        at Object.Module._extensions..js (module.js:550:10)

But, it can work.

    First off, don't worry! You can still run your node application even when seeing this error. Basically, what's happening is: during the initial "npm install" process, the MongoDB module tried to create a couple of files using Python v2.7. If you don't happen to have that installed (even if you have a higher version of Python, it won't work), then it can't build the files, so it falls back on a JavaScript-driven system instead. This system works just fine, but in a production environment with a lot of data handling, it can be slow, so it's better to have the binaries.
    
    If you want to get rid of the error, follow these steps:
    
    Install Python 2.7 (this will not impact existing Python 3.x installs).
    Delete your node_modules directory and everything in it.
    Re-run npm install in your nodetest1 directory from the command line.

My Mac has Python 2.7.10, and I ignore it.

### Add User

##### Input Data page
Add following code above the last module.exports line:

    /**
    * GET New User Page.
    */
    router.get('/newuser', function (req, res) {
      res.render('newuser', {title: 'Add New User'});
    });

Add new user page template.

    
    extends layout

    block content
        h1= title
        form#formAddUser(name='addUser', method="post", action="/adduser")
            input#inputUserName(type="text", placeholder="username", name="username")
            input#inputUserEmail(type="text", placeholder="useremail", name="useremail")
            button#btnSubmit(type="submit") submit
    
We are creating a form with the ID "formAddUser", method is Post, action is adduser.
  
##### Add post function

Need to do is add a route for POSTing to /adduser.

Create our insertion function in `/routes/index.js` and put the above the final module.exports line (it doesn't REALLY matter, but it makes things cleaner to wrap up with the export).
Type following:

    /**
     * POST to Add User Service.
     */
    router.post('/adduser', function (req, res) {
      //Set our internal DB variable
      var db = req.db;
    
      //Get our form values. These rely on the "name" attributes
      var userName = req.body.username;
      var userEmail = req.body.useremail;
    
      //Set our collection
      var collection = db.get('usercollection');
    
      //Submit to the DB
      collection.insert({
        "username":userName,
        "email":userEmail
      }, function (err, doc) {
        if (err){
          //If it failed, return error
          res.send("There was a problem adding the information to the database.");
        }
        else{
          //Add forward to success page
          res.redirect('userlist');
        }
      });
    });

  