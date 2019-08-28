# MERN-IBMCloud




## Getting Started

> As an alternative to the steps below, you can [create this project as a starter kit on IBM Cloud](https://cloud.ibm.com/developer/appservice/create-app?starterKit=6da47a55-cff8-344d-a8ec-08a24c9e1936), which automatically provisions required services, and injects service credentials into a custom fork of this pattern.

Install the latest version of the [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) CLI.

* For macOS and Linux, run the  following command:
  ```
  curl -sL https://ibm.biz/idt-installer | bash
  ```

* For Windows 10 Pro, run the following command in a PowerShell prompt as Admistrator:
  ```
  [Net.ServicePointManager]::SecurityProtocol = "Tls12"; iex(New-Object Net.WebClient).DownloadString('https://ibm.biz/idt-win-installer')
  ```

> *NOTE:* IDT builds and runs the project using Docker containers, the recommended approach for cloud native development. However, direct use of native tools (e.g. npm) is also supported. See the [Appendix](APPENDIX.md) for more information.

## Building your MERN app
// modify
login

`ibmcloud dev create`
Select 2. Backend Service / Web App
4. Node
2. MERN Stack: MongoDB, Express.js, React, Node.js
name
n
3. No DevOps, with manual deployment

https://github.com/IBM/mern-app

#### Working in development mode

1. Build the project with all dependencies, including dev dependencies, with the following command:

    ```
    ibmcloud dev build --debug
    ```    

    > *NOTE:* Ensure a Docker daemon is running before issuing this command.


2. Run the app in dev mode with the command:

    ```
    ibmcloud dev shell run-dev &
    ```

    A web server runs on port `3000` and the app itself runs on port `3100`. The web server and app automatically reload if changes are made to the source.


#### Working in release mode

1. Build the project:

    ```
    ibmcloud dev build
    ```

    This builds the project using `Dockerfile-tools`. Effectively equivalent to `idt build --debug`.

2. Run the project:

    ```
    ibmcloud dev run
    ```
    

## Deploying your MERN app

#### As a Cloud Foundry app

To deploy the app to Cloud Foundry:

```
ibmcloud dev deploy
```

## Start building

http://jamesknelson.com/using-es6-in-the-browser-with-babel-6-and-webpack/

# For ES6/ES2015 support
npm install babel-preset-es2015 --save-dev

# If you want to use experimental ES7 features
npm install babel-preset-stage-0 --save-dev

in webpack.common.js
presets: ['es2015', 'stage-0', 'react'],

npm i bcryptjs is-empty jsonwebtoken passport passport-jwt validator
npm i axios classnames jwt-decode react-redux react-router-dom redux redux-thunk --save-dev
npm i react react-dom --save dev

change all class to className in jsx

Adding Bootstrap (https://reactstrap.github.io/)

First things first, let’s install reactstrap:

$ npm install --save reactstrap react react-dom

create-react-app requires Bootstrap to be installed. Here’s how:

$ npm install bootstrap --save
$ npm install --save reactstrap react react-dom

Next import Bootstrap into your src/index.js file:

import 'bootstrap/dist/css/bootstrap.min.css';

-------------------------------------------
npm i bcryptjs is-empty jsonwebtoken mongoose passport passport-jwt validator

mkdir config && cd config && touch keys.js

module.exports = {
  mongoURI: "YOUR_MONGOURI_HERE" 
};


//server.js
// Add your code here
const mongoose = require("mongoose");
const bodyParser = require("body-parser");

// Bodyparser middleware
app.use(
    bodyParser.urlencoded({
        extended: false
    })
);
app.use(bodyParser.json());

// DB Config
const db = require("./config/keys").mongoURI;

// Connect to MongoDB
mongoose
    .connect(
        db, {
            useNewUrlParser: true
        }
    )
    .then(() => console.log("MongoDB successfully connected"))
    .catch(err => console.log(err));

Setting up our database schema
mkdir models && cd models && touch User.js

// Check if needed
Setting up form validation

mkdir validation && cd validation && touch register.js login.js

//register.js
const Validator = require("validator");
const isEmpty = require("is-empty");

module.exports = function validateRegisterInput(data) {
    let errors = {};

    // Convert empty fields to an empty string so we can use validator functions
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
    }

    return {
        errors,
        isValid: isEmpty(errors)
    };
};

//login.js
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
    }

    return {
        errors,
        isValid: isEmpty(errors)
    };
};

 Setting up our API routes
 mkdir routes && cd routes && mkdir api && cd api && touch users.js

user.js

const express = require("express");
const router = express.Router();
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");
const keys = require("../../config/keys");

// Load input validation
const validateRegisterInput = require("../../validation/register");
const validateLoginInput = require("../../validation/login");

// Load User model
const User = require("../../models/User");

Create the Register endpoint
// @route POST api/users/register
// @desc Register user
// @access Public
router.post("/register", (req, res) => {
    // Form validation
    const { errors, isValid } = validateRegisterInput(req.body);
    
    // Check validation
    if (!isValid) {
      return res.status(400).json(errors);
    }User.findOne({ email: req.body.email }).then(user => {
      if (user) {
        return res.status(400).json({ email: "Email already exists" });
      } else {
        const newUser = new User({
          name: req.body.name,
          email: req.body.email,
          password: req.body.password
        });
        
        // Hash password before saving in database
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
  
  Setup passport
  cd config && touch passport.js

keys.js

module.exports = {
  mongoURI: "YOUR_MONGOURI_HERE", //mongoURI: "mongodb+srv://user:password12341234@cluster0-mgv8n.mongodb.net/user?retryWrites=true&w=majority",
  secretOrKey: "secret"
};

passport.js
const JwtStrategy = require("passport-jwt").Strategy;
const ExtractJwt = require("passport-jwt").ExtractJwt;
const mongoose = require("mongoose");
const User = mongoose.model("users");
const keys = require("../config/keys");

const opts = {};
opts.jwtFromRequest = ExtractJwt.fromAuthHeaderAsBearerToken();
opts.secretOrKey = keys.secretOrKey; module.exports = passport => {
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

Create the Login endpoint
//users.js

// @route POST api/users/login
// @desc Login user and return JWT token
// @access Public
router.post("/login", (req, res) => {
    // Form validation
    const { errors, isValid } = validateLoginInput(req.body);

    // Check validation
    if (!isValid) {
        return res.status(400).json(errors);
    } const email = req.body.email;
    const password = req.body.password;

    // Find user by email
    User.findOne({ email }).then(user => {
        // Check if user exists
        if (!user) {
            return res.status(404).json({ emailnotfound: "Email not found" });
        }

        // Check password
        bcrypt.compare(password, user.password).then(isMatch => {
            if (isMatch) {
                // User matched

                // Create JWT Payload
                const payload = {
                    id: user.id,
                    name: user.name
                };
                // Sign token
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
            }
            else {
                return res
                    .status(400)
                    .json({ passwordincorrect: "Password incorrect" });
            }
        });
    });
});

module.exports = router;


server.js
const passport = require("passport");
const users = require("./routes/api/users");

// Passport middleware
app.use(passport.initialize()); 

// Passport config
require("./config/passport")(passport); 

// Routes
app.use("/api/users", users);


Testing our API routes using Postman

Slides:
Intro to MERN
Quickly go over components
Start with server side (show components needed) with auth
exercise
Frontend react (go over simple react then redux)
Add auth
Deploy to cloud

















    This runs the project using the release image built on the fly using `Dockerfile`. Hot reload is not available in the release image.
