---
title: WEB322 Week 10
layout: default
---

## WEB322 Week 10 Notes

### Managing State Information

Up to this point in the course, we have been building simple applications as examples of how to use the basic building blocks and concepts of a website. However, one of the core concepts that almost every serious website will use has been missing. This is the concept of **state**. One of the most common uses for state is having users and logins on a website. Adding a login and user accounts to a website is not that difficult and it will enable you to secure certain routes and make sure information is only available to be viewed by the correct audience.

There are 3 core pieces required in your applications to be able to deliver this:

1.  Cookies
2.  Sessions
3.  Encryption

We are going to cover Cookies and Sessions in this lecture and briefly touch on encryption as it applies to cookies and sessions. In week 12, we will go into more detail on encryption for other purposes.

<br>

#### Cookies

Cookies are pieces of data that are passed back and forth from the browser to the server that hold state information about the current audience interacting with your application.

Cookies are not set and maintained automatically you have to code this functionality into your application to support it.

As we have learned; each time a request is made from the browser or a response is sent from the server, there is a set of headers attached to the data payload. Request headers contain the ‘Cookie’ header and response headers contain the ‘Set-Cookie’ header. These headers are a string of semi colon separated values that you can refer to in server side code through the ‘req’ object. The most common type of data we want to place in the cookie is a session value.

session contains information about the user, and the session is placed as one of many potential key:value pairs of data in one or more cookies

Here is an example of what a cookie might look like in the request header when inspecting it in the Chrome dev tools

> **Cookie:**_gat=4; _ga=GA1.6.14184624.14881926; session=9HQWRacCvAHDw7KTIw.P_aFNgqKkZrcySj7Jk5HFI8e6TNXBpRemrFczm3YiiVOs7gFG7qZ1GFXJ5Uw-tyOrL6VKd-QoDpYl2INZ1SVjPvt36nq3GOq3kTDh2r9lt580UIl0dNb0Wj_b5IjCOaPnrpqPg-SViSpvQXS7TtRhv6T-FO9UrSRiVWXqt-QBFzLxncbkVgwotroHNRPvjLtdG8ReU34OaD1uQyz5odSpmXlyY_kk.1488044367536.1209600000.M-IJbIdxARUmP-5ummaUOIzmxzWYvbPxawLoBz74

And here is an example of the ‘Set-Cookie’ header in a response from the server to update it after a request

> **Set-Cookie:**session=z80OxktaQTVma-Eclg.QfJOeQ6sTt2g7HfqjpqHlLsuC5ZFYltXmbs_5XuC5Naw-qVuYEItIYXOQZbEwhaEdcR7R35bmnUPLElJuWirq-AUS_K79TO7k9KhXrgUZvQb6gQxa_bcGBoH6onPMhCotDXagGXS2U-MjdsdCo2gtPXpjh0QkPw-oe3-oQr7kj2JidXlSlYweE8sFYsldfBPG3zDMly0MXbhOI4.14880447805.12090000.SRbFKSnQ1-tbEe8NTNxqQHPjnvOhUdC5B4VYs; path=/; expires=Sat, 11 Mar 2017 17:39:28 GMT; secure; httponly

![An example of Cookie and Set-Cookie headers on blackboard](/WEB322-Backup/media/uploads/2016/08/cookies.png)

An example of Cookie and Set-Cookie headers on blackboard



As you can see, the Cookie header contains the session key and value pair but it can also contain other things as well depending on what the website wants to keep in state between requests from the user. You can also see that its just a jumbled up mess. That’s because this particular example uses an encrypted session. Using encrypted sessions is **EXTREMELY** important in a production environment. If you do not encrypt your sessions and cookies then a user can look at the request and response headers and potentially pull out very sensitive information about the user those cookies are meant for. As a general rule, all sessions that are persistent between the client and server should always be encrypted and also always sent over HTTPS as well. We will cover HTTPS encryption in week 12.

Let’s move on to talk more about sessions and implementing them.

<br>

#### Sessions

Implementing sessions is now a fairly simple task in Express.js. The platform has matured and libraries have been tested with thousands of websites and billions of logins. These libraries are easy to use and integrate into your own projects.

The most popular library for implementing client sessions is Mozilla’s client-sessions node library. This library focuses on keeping sessions between the client and server on the client side. This has several advantages over storing the session on the server side and keeping track of it on the server. For example, if the server restarts and you do not save session information in a persistent storage location, the sessions will be lost and all your users will be logged out anytime the server restarts.

It is also beneficial to keep your sessions on the client side (and have them continuously sent with each request), because this enables you to host a web site that has multiple web servers and the session will still be maintained regardless of which server is used to process the response. In an environment where sessions are kept track of on the server and you have multiple web servers that are load balanced, you will need to make sure each users’ requests are always sent to the same web server with each request to preserve their session. Alternatively, you could attempt to replicate server session information between web servers, however this can be very complex. For these reasons, storing the session on the client makes scaling and session management a lot easier.

<br>

#### Authentication vs Authorization

Authentication and Authorization mean two entirely different things. It can be easy to confuse them, so let’s discuss them a little bit before we begin to implement sessions and secure endpoints. This will enable you to be more comfortable explaining and debugging the two different concepts.

Authentication is the answer to “Who are you?”. It involves supplying credentials to identify yourself to the server and establish a session for your user account.

Authorization is the answer to “What do you have access to?”. It involves checking your permissions to resources you have requested and acting accordingly. You may be authenticated with the server and have a user session but you might not be authorized to view a certain resource. (No permissions!)

Here is a great 1 min video that explains it from MongoDB University. MongoDB also has the concept of authentication and authorization.

<iframe title="Authentication vs Authorization" width="100%" height="380" src="https://www.youtube.com/embed/awJHaEKVMKI" frameborder="0" allowfullscreen=""></iframe>

<br>

#### Status codes

There are a number of standard http response status codes that can be used by your application to inform the browser of whether a request was rejected because of an authentication problem or an authorization problem.

*   401 status code (Unauthorized) Authentication error. The resource exists but it requires the user to be authenticated first to view it. It may also require permissions and be checked again after authenticating for proper authorization.

*   403 status code (Forbidden) Authorization error. The resource exists but the user does not have permission to view it.

*   404 status code Not Found. The resource that was requested was not found on the server. This is commonly used when a url is requested that just doesn’t exist.

<br>

#### Adding sessions to an Express.js application

Now that we’ve explained what cookies, sessions, authentication, authorization, and status codes are, we can get to the coding part of the lecture and see how it all comes together.

We will break down the project to add client sessions handling into 4 parts:

1.  Add the client-sessions library to the project.

2.  Create a middleware function to setup the client-sessions configuration.

3.  Write a login route handler to process a POST to /login and establish **authentication** of a user session.

4.  Add a middleware function that handles checking for **authorization** of a user for a specific resource.

<br>

#### Step 1: Add the client-sessions library to the project.

We will need to npm install the client-sessions library and –save it to the package.json file

```bash
npm install client-sessions
```

Then we just have to “require” it in our server file.

```javascript
const express = require("express");
const app = express();
const exphbs = require("express-handlebars");
const clientSessions = require("client-sessions");

const HTTP_PORT = process.env.PORT || 8080;
```

<br>

#### Step 2: Create a middleware function to setup client-sessions.

Now we need to register clientSessions as a middleware and configure it, as well as register handlebars as a view rendering engine and express.urlencoded() to handle our application/x-www-form-urlencoded POST data.

```javascript
// Register handlerbars as the rendering engine for views
app.engine(".hbs", exphbs.engine({ extname: ".hbs" }));
app.set("view engine", ".hbs");

// Setup the static folder that static resources can load from
// like images, css files, etc.
app.use(express.static("static"));

// Setup client-sessions
app.use(clientSessions({
  cookieName: "session", // this is the object name that will be added to 'req'
  secret: "week10example_web322", // this should be a long un-guessable string.
  duration: 2 * 60 * 1000, // duration of the session in milliseconds (2 minutes)
  activeDuration: 1000 * 60 // the session will be extended by this many ms each request (1 minute)
}));

// Parse application/x-www-form-urlencoded
app.use(express.urlencoded({ extended: false }));
```

<br>

#### Step 3: Write a login route handler

Now we need to setup a login route that will check that the username and password match the user who tried to login. We will create login.hbs, and dashboard.hbs templates for the login and dashboard pages.

```javascript
// call this function after the http server starts listening for requests
function onHttpStart() {
  console.log("Express http server listening on: " + HTTP_PORT);
}

// A simple user object, hardcoded for this example
const user = {
  username: "sampleuser",
  password: "samplepassword",
  email: "sampleuser@example.com"
};

// Setup a route on the 'root' of the url to redirect to /login
app.get("/", (req, res) => {
  res.redirect("/login");
});

// Display the login html page
app.get("/login", function(req, res) {
  res.render("login", { layout: false });
});

// The login route that adds the user to the session
app.post("/login", (req, res) => {
  const username = req.body.username;
  const password = req.body.password;

  if(username === "" || password === "") {
    // Render 'missing credentials'
    return res.render("login", { errorMsg: "Missing credentials.", layout: false });
  }

  // use sample "user" (declared above)
  if(username === user.username && password === user.password){

    // Add the user on the session and redirect them to the dashboard page.
    req.session.user = {
      username: user.username,
      email: user.email
    };

    res.redirect("/dashboard");
  } else {
    // render 'invalid username or password'
    res.render("login", { errorMsg: "invalid username or password!", layout: false});
  }
});

// Log a user out by destroying their session
// and redirecting them to /login
app.get("/logout", function(req, res) {
  req.session.reset();
  res.redirect("/login");
});

app.listen(HTTP_PORT, onHttpStart);
```

<br>

#### Step 4: Add a middleware function that checks for authorization

The last thing we need to do is add the dashboard route and add a middleware function to check for authentication before rendering the dashboard.

```javascript
// An authenticated route that requires the user to be logged in.
// Notice the middleware 'ensureLogin' that comes before the function
// that renders the dashboard page
app.get("/dashboard", ensureLogin, (req, res) => {
  res.render("dashboard", {user: req.session.user, layout: false});
});
```

At the top of the file just after we declared the static hardcoded user, we can add a ensureLogin function that is called to check if the user is logged in and redirect them to login if they are not, otherwise continue on to the next middleware in the route handler.

```javascript
// This is a helper middleware function that checks if a user is logged in
// we can use it in any route that we want to protect against unauthenticated access.
// A more advanced version of this would include checks for authorization as well after
// checking if the user is authenticated
function ensureLogin(req, res, next) {
  if (!req.session.user) {
    res.redirect("/login");
  } else {
    next();
  }
}
```

Now we can create the login.hbs and dashboard.hbs files in the /views folder

<br>

#### login.hbs

```handlebars
<!doctype html>
<html>

<head>
  <title>Login</title>
</head>

<body>
  {% raw %}{{#if errorMsg}}{% endraw %}
    <h5>{% raw %}{{errorMsg}}{% endraw %}</h5>
  {% raw %}{{else}}{% endraw %}
    <h5>Please login below</h5>
  {% raw %}{{/if}}{% endraw %}
  <form method="post" action="/login">
    <label>Username</label><input name="username" type="text">
    <label>Password</label><input name="password" type="password">
    <button type="submit">Login</button>
  </form>
</body>

</html>
```
<br>

#### dashboard.hbs

```handlebars
<!doctype html>
<html>

<head>
  <title>Dashboard</title>
</head>

<body>
  <h3>Hello {% raw %}{{user.username}}{% endraw %}</h3>
  <p>Welcome to your dashboard</p>
  <p>Here is the information we have on file for you:</p>
  <h4>username: {% raw %}{{user.username}}{% endraw %}</h4>
  <h4>email: {% raw %}{{user.email}}{% endraw %}</h4>
  <a href="/logout">Logout</a>
</body>

</html>
```

<br>

### Sources

*   [Mozilla’s client-sessions library](https://github.com/mozilla/node-client-sessions)
*   [passport module](https://github.com/jaredhanson/passport)
*   [Using secure client-side sessions to build simple and scalable node.js applications](https://hacks.mozilla.org/2012/12/using-secure-client-side-sessions-to-build-simple-and-scalable-node-js-applications-a-node-js-holiday-season-part-3/)
*   [Everything you ever wanted to know about node.js sessions](https://stormpath.com/blog/everything-you-ever-wanted-to-know-about-node-dot-js-sessions)