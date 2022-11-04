
# NextJS

One virtual hosting service making big waves in the web development community is [NextJS](https://vercel.com/) (a suite of tools run by Vercel).

For this assignment, we'll be making a public website without writing any server code. Like GitHub Pages, it will read from your github project and deploy accordingly. However, it's set of tooling is much more complete than GitHub (for instance, NextJS provides serverless functions).

The assignment is to make a starter website, put your Clemson Username on the page, and then post a link to your website in Canvas.

1. [Sign Up to Vercel](https://vercel.com/signup) (using GitHub is the best option).
2. Select the Next.js Boilerplate from the available [templates](https://vercel.com/templates).
3. Press Deploy
4. Create a new GitHub Git Repository. At this point, it will ask for permission to read and modify your repositories. If you feel uncomfortable providing full access, you can limit it to a selection of repositories (you can select your previous assignments' repository).
5. Give your project a name and deploy it!
6. Visit the link when deployment is finished and see that it renders.
7. Edit pages/index.js in your repo to include your clemson username (for instance, by removing "Get started by editing ..." and replacing it with "Page owned by clemsonusername.")
8. Have fun playing with your website!

Note that NextJS is generally built on top of React. So your javascript includes an html-in-javascript syntax called [jsx](https://reactjs.org/docs/introducing-jsx.html). JSX is not true javascript, which means this NextJS uses a [transpiler](https://babeljs.io/) that converts your project code to something the browser would understand.

We will talk about React's model in future classes, but for now you can just imagine that it's true HTML in the page and edit accordingly.

And that's it. We have now created a new website. NextJS is a powerful framework that makes creating a website easy, but customizing the stack for various purposes more challenging. For many uses though, Vercel might just be the right approach for your professional projects.





