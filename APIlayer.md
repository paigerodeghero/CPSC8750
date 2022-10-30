
# RESTful Gateway

One of the features of REST is that following its constraints allows for layering.

We're going to implement a [gateway](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_3) between the web browser (which expects HTML) and the [Open Trivia DB](https://opentdb.com) (which serves JSON).

We'll do this by expanding on the Express.js server we've made from previous assignments.

## The Content Server

The Open Trivia DB provides a [free API](https://opentdb.com/api_config.php) for loading up random trivia questions.

The request will be an HTTP GET request to a url, and the response will be some JSON.

Here's an example request for three questions:
```js
// result of https://opentdb.com/api.php?amount=3
{
  "response_code":0,
  "results":[
    {
      "category":"Entertainment: Music",
      "type":"multiple",
      "difficulty":"easy",
      "question":"Which of these is NOT the name of an album released by English singer-songwriter Adele?",
      "correct_answer":"12",
      "incorrect_answers":[
        "19",
        "21",
        "25"
      ]
    },
    {
      "category":"Entertainment: Video Games",
      "type":"boolean",
      "difficulty":"easy",
      "question":"Luigi is taller than Mario?",
      "correct_answer":"True",
      "incorrect_answers":["False"]
    },
    {
      "category":"Geography",
      "type":"multiple",
      "difficulty":"easy",
      "question":"Greenland is a part of which kingdom?",
      "correct_answer":"Denmark",
      "incorrect_answers":[
        "Sweden",
        "Norway",
        "United Kingdom"
      ]
    }
  ]
}
```

We'll use the API as the content server, and our code will just be a translation layer.

## Installing Dependencies

The Javascript standards committees have defined a [standard api for web requests](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), however, Node [only recently](https://blog.logrocket.com/fetch-api-node-js/) added it behind a flag.

So, we're going to use a library. In your project, run:
```
npm install node-fetch@2
```

Then add the import at the top of your server's implementation file, add:
```js
const fetch = require('node-fetch');
```

## Make an Endpoint

In your server code, let's add a "/trivia" endpoint:

```js
app.get("/trivia", (req, res) => {
  res.send('TODO');
});
```

To see the page in action, run `npm start` in your terminal and visit http://localhost:3000/trivia

You should be able to see the word "TODO" in your browser.

## Calling to the Backend Server

The fetch api is _asynchronous_, meaning that it takes time to fetch the data and we do not want our entire server to lock up while the network requests are occuring.

This means we will now use the [async/await syntax](https://javascript.info/async-await).

Change the "/trivia" endpoint to the following:
```js
app.get("/trivia", async (req, res) => {
  // fetch the data
  const response = await fetch("https://opentdb.com/api.php?amount=1&type=multiple");

  // interpret the body as json
  const content = await response.json();

  // TODO: make proper html
  const format = JSON.stringify(content, 2);

  // respond to the browser
  // TODO: make proper html
  res.send(JSON.stringify(content, 2));
});
```

Restarting the server and visiting the webpage should output the raw json request.

Let's quickly add some very basic error handling that passes the issue onto the user:
```js
app.get("/trivia", async (req, res) => {
  // fetch the data
  const response = await fetch("https://opentdb.com/api.php?amount=1&type=multiple");

  // fail if bad response
  if (!response.ok) {
    res.status(500);
    res.send(`Open Trivia Database failed with HTTP code ${response.status}`);
    return;
  }

  // interpret the body as json
  const content = await response.json();

  // fail if db failed
  if (content.response_code !== 0) {
    res.status(500);
    res.send(`Open Trivia Database failed with internal response code ${content.response_code}`);
    return;
  }

  // respond to the browser
  // TODO: make proper html
  res.send(JSON.stringify(content, 2));
});
```

## Serving HTML

Make a new file called `trivia.ejs` (your project should have a `welcome.ejs` view already) with the following content:
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Trivia</title>
    <link rel="stylesheet" href="app.css">
  </head>
  <body>
    <header>
      <h1>Trivia Question</h1>
    </header>
    <p><%- question %></p>
    <ol>
      <% answers.forEach(answer => { %>
        <li><%- answer %></li>
      <% }); %>
    </ol>
    <footer>
      <dl>
        <dt>Category</dt>
        <dd><%- category %></dd>
        <dt>Difficulty</dt>
        <dd><%- difficulty %></dd>
      </dl>
    </footer>
  </body>
</html>
```

Now replace the `res.send(JSON.stringify(content, 2))` call with a call to `res.render('trivia', {...})`.

The render call will need values passed into it, it is up to you to take the trivia question and pass in its parts.

## Adding Links

We now have a question rendered; however, it doesn't present anything a user can do with it. Let's allow them to pick an answer.

Expand your code so that the answers aren't just text, but also links that pop-up.

Here's some base-line code to inspire you:
```js
const correctAnswer = ...;
const answers = ...;

const answerLinks = answers.map(answer => {
  return `<a href="javascript:alert('${
    answer === correctAnswer ? 'Correct!' : 'Incorrect, Please Try Again!'
    }')">${answer}</a>
  }`
})
```

When you have an implementation that shows the correct/incorrect alerts on press, please submit your GitHub link to Canvas.

Homework will be assigned to further this exercise. TBA.
