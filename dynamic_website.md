
# Server-Side Dynamic Website

In the previous assignment, you made a website using Express.js and Node.
Today, we're going to expand on that site.

> It is ok if you still do not have do not have fly.io, this lab can be done on localhost.

## Serving HTML

We left off with a website that merely said "Hello, World!".
However, we actually did not serve a true html document.
Instead, we just sent the string "Hello, World!" and the browser ["helpfully"](https://developer.mozilla.org/en-US/docs/Web/HTML/Quirks_Mode_and_Standards_Mode) renders it.

Last time we left off, we had code like this:
```js
// The main page of our website
app.get('/', (req, res) => {
  res.send('Hello World!')
});
```

Edit your file to now send true html.
```js
app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <title>An Example Title</title>
      </head>
      <body>
        <h1>Hello, World!</h1>
        <p>HTML is so much better than a plain string!</p>
      </body>
    </html>
  `);
});
```

> Note: The backticks are a way of specifying [multiline strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) in javascript.

Great! Express.js will tell the browser by default that content is html, so we can see a functioning webpage by running `npm start` in your terminal and then opening `http://localhost:3000/`

As a reminder, pressing ctrl-C will kill off your server and return control of your terminal.

## Serving Static Files

Now that we've got html, we can actually reference other files from the webpage.
This makes it a good time to add CSS.

Make a new "public" folder in your project.
If you're new to command-line, you can do this with:

```
mkdir public
```

This folder is where we'll store files that we want exposed to the outisde world (hence the name "public").

To tell your server code about the public folder, add a line of code to your app:
```js
app.use(express.static('public'));
```

Now, let's make a css file "app.css" in your public folder.

```css
body {
  background-color: black;
  color: white;
}
```

And of course, we now have to reference that css file in our html:
```js
app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <title>An Example Title</title>
        <link rel="stylesheet" href="app.css">
      </head>
      <body>
        <h1>Hello, World!</h1>
        <p>HTML is so much better than a plain string!</p>
      </body>
    </html>
  `);
});
```

Now if you run your server, you should see styled html.

You can look at the network tab to see that express will send the css down with the header:
```
Content-Type: text/css; charset=UTF-8
```

Express knows it's css because the file ends with `.css`.
There's a number of built-in filetypes.

> Note!
> You may make a change to the css and discover that the browser won't update.
> That can be due to caching, you may need to run a [shortcut](https://fabricdigital.co.nz/blog/how-to-hard-refresh-your-browser-and-clear-cache) to force the browser to reload.
> On Safari it can be accomplished by pressing the refresh button while holding shift.

## Dynamic HTML

Many websites nowadays are heavily javascript based, and sometimes don't even show content until the js loads.

However, the more old-school approach (and it can still be a faster user experience) is to have the server build out the initial html and then have the js fill in some details.

We are going to use server-side rendering for this assignment.

First, let's do an example of what not to do (because almost everyone makes this mistake).

### Direct Injection, (aka. The Wrong Way)

Let's directly make dynamic html:
```js
app.get('/', (req, res) => {
  // reads the url parameter
  // http://domain/?name=text
  const name = req.query.name || "World";

  res.send(`
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <title>An Example Title</title>
        <link rel="stylesheet" href="app.css">
      </head>
      <body>
        <h1>Hello, ${name}!</h1>
        <p>HTML is so much better than a plain string!</p>
      </body>
    </html>
  `);
});
```

Now if you run the server and navigate to `http://localhost:3000/?name=Web%20Dev` you should get a website that says "Hello, Web Dev!".

> If you are unfamiliar with url encoding, the `%20` in the url is a space.
> This is called [percent encoding](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding)

If you look at the source tab in your browser, you'll see that the browser received th raw html:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>An Example Title</title>
    <link rel="stylesheet" href="app.css">
  </head>
  <body>
    <h1>Hello, Web Dev!</h1>
    <p>HTML is so much better than a plain string!</p>
  </body>
</html>
```

Great, but what happens if the user set their name to `<script>alert("Malicious Code!")</script>`?

Try it yourself by visiting `http://localhost:3000/?name=%3Cscript%3Ealert(%22Malicious%20Code%22)%3C%2Fscript%3E`.

Even more fun, what happens if the user sets their name to:
```
<script>while(true) alert("Malicious Code!")</script>
```

You can try it yourself (if you dare!) by visiting `http://localhost:3000/?name=%3Cscript%3Ewhile(true)%20alert(%22Malicious%20Code!%22)%3C%2Fscript%3E`

So what's happening?
The issue is that the server sends the browser this html:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>An Example Title</title>
    <link rel="stylesheet" href="app.css">
  </head>
  <body>
    <h1>Hello, <script>while(true) alert("Malicious Code!")</script>!</h1>
    <p>HTML is so much better than a plain string!</p>
  </body>
</html>
```

That script tag is parsed _as html_ instead of a plain string.
Which means, the browser runs this code and you end up in an infinite loop of pop-ups.

An attack like this is called "[code injection](https://en.wikipedia.org/wiki/Code_injection)" or "[Cross-Site Scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting)"

This kind of mistake is super easy to make.
In fact, it practically makes PHP insecure by default.
This vulnerability is why [all myspace users declared Samy their hero](https://www.youtube.com/watch?v=DtnuaHl378M) in 2005.

Technically, this can be sanitized by running something like [html-entities](https://www.npmjs.com/package/html-entities)' encode method:
```js
const {encode} = require('html-entities');

// ... snipped out code ...

app.get('/', (req, res) => {
  // reads the url parameter
  // http://domain/?name=text
  const name = req.query.name || "World";

  res.send(`
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <title>An Example Title</title>
        <link rel="stylesheet" href="app.css">
      </head>
      <body>
        <h1>Hello, ${encode(name)}!</h1>
        <p>HTML is so much better than a plain string!</p>
      </body>
    </html>
  `);
});
```

However, this is still incredibly easier to forget to do.

The web community has actually solved this problem _architecturally_, by not injecting raw strings into html.

React, for instance, makes it really hard to use raw html.
Another way to do this is with a template engine.

### Template Engine

Add [ejs](https://www.npmjs.com/package/ejs) as a dependency:

```
npm install ejs
```

And add to the javascript file:
```js
// set the view engine to ejs
app.set('view engine', 'ejs');
```

Make a new folder called "views" with a file called "welcome.ejs" with this content:
```ejs
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>An Example Title</title>
    <link rel="stylesheet" href="app.css">
  </head>
  <body>
    <h1>Hello, <%= name %>!</h1>
    <p>HTML is so much better than a plain string!</p>
  </body>
</html>
```

This ejs syntax is a hybrid of javascript and html.
The `<%= name %>` field tells it to put the "name" parameter into that position (but made to be html-safe!).

We then tell express to serve the template:
```js
app.get('/', (req, res) => {
  res.render('welcome', {
    name: req.query.name || "World",
  });
});
```

Now, on your own, modify the code to print out `new Date().toLocaleString()` so that the output looks something like this:

```
Hello, World!

This site was accessed at 10/7/2022, 8:30:57 AM
```

## Cookies

We can write a cookie easily in express js.
Add this line to your `app.get('/', ...)` code:
```js
res.cookie('visited', Date.now().toString());
```

We've now set the "visited" cookie to the timestamp (in ms) that the page was requested.

If you look at the network tab, you can see the response came with the cookie.

Let's add a second example cookie, "visitorId," so that your javascript file looks something like this:

```js
const express = require('express');

//... code ...

let nextVisitorId = 1;

// the main page
app.get('/', (req, res) => {
  res.cookie('visitorId', nextVisitorId++);
  res.cookie('visited', Date.now().toString());
  res.render('welcome', /* params */)
});

```

While express makes setting cookies relatively easy, it doesn't by default come with a good cookie parser.
You can see how the browser sends multiple cookies in one string by adding `console.log(req.headers.cookie)` to the code.

To add cookie parsing, run
```
npm install cookie-parser
```

Then import `cookieParser` by putting at the top:
```js
const cookieParser = require('cookie-parser');
```

And after you instantiate your `app` variable, add:
```js
app.use(cookieParser());
```

Now our request will come with the cookies parsed!
Try adding `console.log(req.cookies)` to the request to have your server print out the parsed cookies into the terminal.

Now, on your own, read from the cookies field and output html that says both:
- How long it's been (in seconds) since they loaded the page (or the text "you have never visited")
- They're visitor id

Make it so that when the browser refreshes the page, it will keep the same visitor id.

In other words, the remainder of your assignment is to try and match [this example site](https://clemson-prodegh.fly.dev/)

## Submitting Assignment
Commit your code to a github repo.
Go to the Dynamic Webpage Assignment in Clemson Canvas and copy the link over for your github repo.

## Addendum

This is an assignment to try to get used to the idea of cookies, but this is not a good way to use cookies.

For instance, in Safari's storage tab in the inspector, you can edit the visitorId cookie to be whatever you want.

In the real world, user ids are probably not going to sequential numbers (because that's guess-able and it reveals information, like Zuckerberg being the 4th user on Facebook) and authentication is not going to be based on a user-readable cookie.

If your cookies have critical information that is allowed to be public, you could potentially sign the cookie so that it is public but cannot be generated externally.
cookie-parser has support for this.

