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

    db.usercollection.find().pretty()db.usercollection.find().pretty()
    
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

Add router in `index.js`, and get `db` by `req`, 