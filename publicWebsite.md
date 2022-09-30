# Making a Public Website with Express.js

We've covered, at a high-level, the MERN stack; this lab is about running through the -E-N sections in practice.

## MER(N): Node.js

People rarely write javascript apps from scratch. Instead, they use a vast collection of (mostly) open-source libraries available through the "Node Package Manager" (npm).

In order to locally run npm, you'll need node itself. Fortunately, node comes with [installers for most major platforms](https://nodejs.org/en/download/package-manager/). If you use mac, we'd recommend downloading [homebrew](https://brew.sh) and then running `brew install node` in your terminal.

Once going through the installers, open a terminal and run `node --version`. If everything was successfully installed, node should log its version number to the scene.

This process should also have installed npm, which you can confirm by running `npm --version`

## Starting Your Project

Make a new public github project titled "clemsonUsername-cpsc8750" (swapping "clemsonUsername" for your Clemson username). It would be helpful when creating it to set the .gitignore file option to "Node."

Download the project locally and open the folder in your terminal.

Run `npm init`. This will take you through a series of question prompts in the terminal. The default answer (the value in parentheses) is good enough for the package name, version, and description. To select the default answer, simply hit enter.

Set your entrypoint to src/index.js (we will make that file in a later assignment).

Otherwise, fill in whatever you like for the remaining questions (you can leave the "test command" response empty, as we will not have tests for this program).

You should see there is a new package.json configuration file that has been added to the repo. You have now initialized a node-package-manager enabled project!

## M(E)RN: Express.js

Express.js is a server library written for node environments. To add it to your project, open your project folder in your terminal and run `npm install express`.

You should see the command run and download express, along with all its dependencies (and all its dependencies' dependencies). It is now usable within the codebase!

Create a folder in your project to hold your javascript source called `src`. Create a new file in that folder called `server.js`. This can be done in the command line with the following three commands:
```
mkdir src
touch src/server.js
open src/server.js
```

Copy this code into server.js:
```js
// use the express library
const express = require('express');

// create a new server application
const app = express();

// Define the port we will listen on
// (it will attempt to read an environment global
// first, that is for when this is used on the real
// world wide web).
const port = process.env.PORT || 3000;


// The main page of our website
app.get('/', (req, res) => {
  res.send('Hello World!')
});

// Start listening for network connections
app.listen(port);

// Printout for readability
console.log("Server Started!");
```

Now run the code by typing `node src/server.js`. Hopefully, it will print out "Server Started!".

You can see your website in action by going to your web browser and visiting http://localhost:3000/

Congratulations, you've just made a working web server!

To kill your server, go to the terminal it's running in and press control+C. That sends a signal to kill the active job.

Commit your code.

## External Hosting

Now that we have this program working on localhost, we're going to move that code onto an actual host that the world will have access to.

We are going to use [fly.io](https://fly.io) which allows accounts to host up to 2 low-traffic websites for free.

To start, you will need to install `flyctl`, [their command-line interface](https://fly.io/docs/hands-on/install-flyctl/). You will also need to register an account by running:
```
flyctl auth signup
```

You do not need to input a credit card, when asked just click the link for trying fly.io for free.

Navigate to your repository within a terminal. We are going to run commands within the project.

But first, in order to make fly.io aware of how to run our project, we will need to add a "start" command.

Open up package.json. In the "scripts" object, add a "start" field with value "node src/server.js". The resulting package.json should look something like this:
```
{
  "name": "some-project-name",
  "version": "1.0.0",
  "description": "",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.1"
  }
}
```

Great, we have added the "start" command. From now on, instead of manually typing `node src/server.js` we can run `npm start` when we wish to run the server locally.

Now fly.io has enough information to engage with their system. Run the `flyctl launch` command, and choose the name `clemson-clemsonUsername`.

Choose any region to host your code, they all work, but the closer the region is to you, the faster it will load your webpage.

Do not set up a Postgresql database at this point. (type no, or just n).

Select deploy now! (type yes, or just y).

Note that fly.io can take a while to get launched, but if all goes correctly, you now have a website with a custom js server in the world-wide-web! To visit your website, run `flyctl open`.

If you get stuck at any point setting up fly.io, or want to experiment with it on your own, they provide a nice [tutorial](https://fly.io/docs/languages-and-frameworks/node/) that gives additional information.
