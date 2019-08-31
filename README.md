This repository walks through building a minimalistic app using the MERN stack, where you will use MongoDB for the database, Express & NodeJS for the backend, and React for the frontend. Redux will be used with React for state management of the generated React components.

MongoDB: A document-based open source database

Node.js: Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js brings JavaScript to the server

Express: A Fast, unopinionated, minimalist web framework for Node.js

React: A JavaScript front-end library for building user interfaces

The user will be able to login/regsiter to access their personal health journal. They will also stay logged in when they close/refresh the page. You will be able to create a new posts as well as read a post.


# Pre-requites
1. [IBM Cloud](http://ibm.biz/devday2) account
2. An account on [MongoDB Atlas](https://cloud.mongodb.com)
1. Install [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools)
2. Install [Docker Engine (Community)](https://docs.docker.com/install/)
    For Windows 10 Home, follow the instructions [here](https://docs.docker.com/toolbox/toolbox_install_windows/)
3. Install [Visual Studio Code](https://code.visualstudio.com/download)
4. Install [NodeJS](https://nodejs.org/en/download/)
5. Install [Postman](https://www.getpostman.com/downloads/)
6. Add [React Developer Tools Chrome Extension](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
7. Add [Redux DevTools Chrome Extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)

# Part 1: Setting Up the Project

Login to IBM Cloud using the CLI command
```
$ ibmcloud login
```
Target Cloud Foundry org/space interactively
```
$ ibmcloud target --cf
```
Target the appropriate resource group (it is likely to be `Default`)
```
$ ibmcloud target -g RESOURCE_GROUP
```
If you are not sure what are the available resource groups, you can run 
```
$ ibmcloud resource groups
```
You can either use the create a MERN application project as a [starter kit on IBM Cloud](https://cloud.ibm.com/developer/appservice/create-app?starterKit=6da47a55-cff8-344d-a8ec-08a24c9e1936) or create a MERN app in the current directory using [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) with the following command
```
$ ibmcloud dev create
```
Select the application type as a Backend Service / Web App, the language as Node and the Starter Kit as MERN Stack. At this stage, don’t select a service to add to the application and, for DevOps toolchain and target runtime environment options, select to have No DevOps, with manual deployment.

Working in development mode
Build the project with all dependencies, including dev dependencies, with the following command:
```
ibmcloud dev build –debug
```    
> *NOTE:* Ensure a Docker daemon is running before issuing this command.
Run the app in dev mode with the command:
```
ibmcloud dev shell run-dev &
```
A web server runs on port `3000` and the app itself runs on port `3100`. The web server and app automatically reload if changes are made to the source.

Working in release mode
Build the project:
```
ibmcloud dev build –debug
```    
This builds the project using `Dockerfile-tools`. Effectively equivalent to `idt build --debug`.
Run the project:
```
ibmcloud dev run
```

# Part 2: Setting Up the Backend

Before we start, check if NodeJS is installed on your system
$ node -v
If the command does not return the node version, make sure to [install it](https://nodejs.org/en/download/)

Open VSCode (Visual Studio Code) and navigate to the newly created project. You can also do that via the terminal by changing to the appropriate directory and typing
$ code .
I recommend going to extensions in VSCode and adding the following extensions (this is optional):
- Prettier - Code formatter (to seamlessly format our Javascript)
- ES7 React/Redux/React-Native/JS snippets (provides you JavaScript and React/Redux snippets in ES7 with Babel plugin features)
Install the following dependencies using npm
```
$ npm i bcryptjs is-empty jsonwebtoken mongoose@5.4.20 passport passport-jwt validator
```
express, mongoose and body-parser were already when the project was created. Mongoose was however added above as we need to upgrade the package version to be able to use it to access a free cluster on [MongoDB Atlas](https://cloud.mongodb.com).

## Create a new MongoDB Deployment
To set up our database, login to [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login). 
Create a new cluster by selecting AWS as the Cloud Provider and create a free tier cluster by selecting a region with FREE TIER AVAILABLE.

Next, create database user by going to SECURITY -> Databse Access and click + ADD NEW USER. Make sure to give your user Read and write to any database privileges. Take note of username & password you entered as you will need it later on in your code to connect to your database.

Now, let’s go to whitelisting your IP address. Go to SECURITY → Network Access and click on ALLOW ACCESS FROM ANYWHERE and confirm.

Finally, to create a new database, go to ATLAS → Clusters and click COLLECTIONS under Cluster0 (the cluster name in this case). Click Add my own data and select Create a new Database. Give it a new DATABASE NAME & COLLECTION NAME and click on Create.

Find your MongoDB URI by clicking on Click CONNECT. After selecting the Connect Your Application option, you should be able to get a connection string should be similar to mongodb+srv://<username>:<password>@cluster0-mgv8n.mongodb.net/<dBName>.
    
Replace <username> and <password> with the database user credentials you just created and <dBName> with the database name.

## Cleaning up
From this point forward in this section, we will be working with the server folder.

Head back to your project in VSCode and go to server folder in your project and delete the comments.js under the model folder, & mongo.js under the routers folder. Then, go to index.js under routers and comment out or remove the following line
```
require('./mongo')(app, server);
```

Open the terminal and type the following to get into the server folder
```
$ cd server
```

Create a config directory and within it a keys.js file
```
$ mkdir config && cd config && touch keys.js
```
Within your keys.js file, add
```
module.exports = {
  mongoURI: "<MONGOURI>" 
};
```
## Setting up our database schema
To define the schema, we will create a model folder. The schema will represent the user. In the terminal, type
```
$ mkdir models && cd models && touch User.js 

y.js
```
In the User.js file, add
```
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

// Create Schema
const UserSchema = new Schema({
  name: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true
  },
  password: {
    type: String,
    required: true
  },
  date: {
    type: Date,
    default: Date.now
  }
});
module.exports = User = mongoose.model("users", UserSchema);
```

## Setting up form validation
Before setting up the routes, let’s create a directory for input validation.
``` 
$ mkdir validation && cd validation && touch register.js login.js
```
Place the following in register.js
```
const Validator = require("validator");
const isEmpty = require("is-empty");

module.exports = function validateRegisterInput(data) {
  let errors = {};// Convert empty fields to an empty string so we can use validator functions
  data.name = !isEmpty(data.name) ? data.name : "";
  data.email = !isEmpty(data.email) ? data.email : "";
  data.password = !isEmpty(data.password) ? data.password : "";
  data.password2 = !isEmpty(data.password2) ? data.password2 : "";
// Name checks
  if (Validator.isEmpty(data.name)) {
    errors.name = "Name field is required";
  }
// Email checks
  if (Validator.isEmpty(data.email)) {
    errors.email = "Email field is required";
  } else if (!Validator.isEmail(data.email)) {
    errors.email = "Email is invalid";
  }
// Password checks
  if (Validator.isEmpty(data.password)) {
    errors.password = "Password field is required";
  }
if (Validator.isEmpty(data.password2)) {
    errors.password2 = "Confirm password field is required";
  }
if (!Validator.isLength(data.password, { min: 6, max: 30 })) {
    errors.password = "Password must be at least 6 characters";
  }
if (!Validator.equals(data.password, data.password2)) {
    errors.password2 = "Passwords must match";
  }return {
    errors,
    isValid: isEmpty(errors)
  };
};
```
Place the following in login.js
```
const Validator = require("validator");
const isEmpty = require("is-empty");
module.exports = function validateLoginInput(data) {
  let errors = {};
// Convert empty fields to an empty string so we can use validator functions
  data.email = !isEmpty(data.email) ? data.email : "";
  data.password = !isEmpty(data.password) ? data.password : "";
// Email checks
  if (Validator.isEmpty(data.email)) {
    errors.email = "Email field is required";
  } else if (!Validator.isEmail(data.email)) {
    errors.email = "Email is invalid";
  }
// Password checks
  if (Validator.isEmpty(data.password)) {
    errors.password = "Password field is required";
  }return {
    errors,
    isValid: isEmpty(errors)
  };
};
```
## Setting up API routes
Go the routers folder and create a users.js file for registration and login
```
$ cd routers && touch users.js
```
Add the following to users.js
```
"use strict";

var mongoose = require("mongoose");
var bodyParser = require("body-parser");
var express = require("express");

const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const keys = require("../config/keys");
const passport = require("passport");

var User = require("../model/User");

module.exports = function(app) {
  var router = express.Router();

  // Load input validation
  const validateRegisterInput = require("../validation/register");
  const validateLoginInput = require("../validation/login");

  // set up other middleware
  app.use(bodyParser.urlencoded({ extended: true }));
  app.use(bodyParser.json());

  // DB Config
  const db = require("../config/keys").mongoURI;

  mongoose
    .connect(db + "user", { useNewUrlParser: true })
    .then(() => console.log("MongoDB successfully connected - user"))
    .catch(err => console.log(err));

  // Passport middleware
  app.use(passport.initialize()); // Passport config
  require("../config/passport")(passport);

  // @route POST api/users/register
  // @desc Register user
  // @access Public
  router.post("/register", (req, res) => {
    // Form validation
    const { errors, isValid } = validateRegisterInput(req.body); // Check validation
    if (!isValid) {
      return res.status(400).json(errors);
    }
    User.findOne({ email: req.body.email }).then(user => {
      if (user) {
        return res.status(400).json({ email: "Email already exists" });
      } else {
        const newUser = new User({
          name: req.body.name,
          email: req.body.email,
          password: req.body.password
        }); // Hash password before saving in database
        bcrypt.genSalt(10, (err, salt) => {
          bcrypt.hash(newUser.password, salt, (err, hash) => {
            if (err) throw err;
            newUser.password = hash;
            newUser
              .save()
              .then(user => res.json(user))
              .catch(err => console.log(err));
          });
        });
      }
    });
  });

  // @route POST api/users/login
  // @desc Login user and return JWT token
  // @access Public
  router.post("/login", (req, res) => {
    // Form validation
    const { errors, isValid } = validateLoginInput(req.body); // Check validation
    if (!isValid) {
      return res.status(400).json(errors);
    }
    const email = req.body.email;
    const password = req.body.password; // Find user by email
    User.findOne({ email }).then(user => {
      // Check if user exists
      if (!user) {
        return res.status(404).json({ emailnotfound: "Email not found" });
      } // Check password
      bcrypt.compare(password, user.password).then(isMatch => {
        if (isMatch) {
          // User matched
          // Create JWT Payload
          const payload = {
            id: user.id,
            name: user.name
          }; // Sign token
          jwt.sign(
            payload,
            keys.secretOrKey,
            {
              expiresIn: 31556926 // 1 year in seconds
            },
            (err, token) => {
              res.json({
                success: true,
                token: "Bearer " + token
              });
            }
          );
        } else {
          return res
            .status(400)
            .json({ passwordincorrect: "Password incorrect" });
        }
      });
    });
  });

  app.use("/api/users", router);
};

```
In your config directory, create a passport.js file.
```
$ cd config && touch passport.js
```
Before we setup passport, let’s add the following to our keys.js file.
```
module.exports = {
  mongoURI: "<MONGOURI>",
  secretOrKey: "secret"
};
```
Place the following in our passport.js
```
const JwtStrategy = require("passport-jwt").Strategy;
const ExtractJwt = require("passport-jwt").ExtractJwt;
const mongoose = require("mongoose");
const User = mongoose.model("users");
const keys = require("../config/keys");const opts = {};
opts.jwtFromRequest = ExtractJwt.fromAuthHeaderAsBearerToken();
opts.secretOrKey = keys.secretOrKey;module.exports = passport => {
  passport.use(
    new JwtStrategy(opts, (jwt_payload, done) => {
      User.findById(jwt_payload.id)
        .then(user => {
          if (user) {
            return done(null, user);
          }
          return done(null, false);
        })
        .catch(err => console.log(err));
    })
  );
};

```
Go to index.js under the routers folder and add the following to the exports
```
require("./users")(app, server);
```

# Part 3: Setting Up the Frontend (React + Redux) & Connecting it to the Backend
Now, we will start working on the React part of our application.

Install the following dependencies using npm
- For ES6/ES2015 support
```
$ npm install babel-preset-es2015 --save-dev
```

- If you want to use experimental ES7 features
```
$ npm install babel-preset-stage-0 --save-dev
```
In the projects parent directory, modify presets in webpack.common.js as follows
```
presets: ['es2015', 'stage-0', 'react'],
```
## Install the rest of the dependencies we need
```
$ npm i axios classnames jwt-decode react-redux react-router-dom redux redux-thunk
```
## Next Steps
For the react + redux part of the application, copy the content of the content of app folder from this repository and paste into your project


# Part 4: Deploy to IBM Cloud
To deploy the app to Cloud Foundry:

```
$ ibmcloud dev deploy
```

