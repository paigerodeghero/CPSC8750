# Cookies

The internet is by default (mostly) anonymous.

Any computer can open up youtube.com.
So how does YouTube keep track of what account is signed in on your computer or, if you're not logged in, what videos you've seen on that computer?

The answer is (primarily) by using cookies.

To explain cookies, we need to look at an HTTP request.
If you open your network tab on your websites and load the page, you will see something like:

```
GET /
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.5 Safari/605.1.15
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Accept-Encoding: gzip, deflate, br
Host: mysite.fly.dev

```

Surprisingly HTTP requests are just text, rather than some complicated binary format (although HTTP3 uses a more efficient format, but it is still not fully supported).

That format is fundamentally:

```
GET /path/to/page.html

```

However, we rarely see the raw GET field by itself.
Instead, the request comes with headers.

A header is a line of text formatted like this:
```
Header-Name: header value
```
Side Note: By convention, headers are usually capitalized, however servers are supposed to treat capitalized and uncapitalized text the same.

The headers in the example request are:
1. User-Agent: The browser that requested the text, for historical reasons, almost all browsers start by claiming they're Mozilla version 5. The string is obnoxiously opaque.
2. Accept: The type of content that the browser expects to receive from this GET request
3. Accept-Language: The user's preferred languages, with weights to help the server decide which it could support
4. Connection: The "Connection: keep-alive" header is a way to hint that the underlying tcp connection to the server should stay open (which can then speed up downloading the js/css that the page references)
5. Accept-Encoding: To lessen the amount of data sent over the network, browsers can declare that they accept compressed files
6. Host: The domain of the website

The server's response looks very similar. It starts with a status code (200 for OK), presents some meta-data headers, and then the actual content.
Here is what the response should look like from your fly.io websites:

```
200
Content-Type: text/html; charset=utf-8
Via: 2 fly.io
Date: Fri, 30 Sep 2022 06:26:52 GMT
Content-Encoding: gzip
ETag: W/"16-MxF3VLoPwDKF0ievLBXQEjIjzKw"
Server: Fly/54d1d930f (2022-09-30)
x-powered-by: Express
fly-request-id: 01GEE8D86XET662W2T18NZN9G8-cdg

... the webpage content, gzipped in this case ...
```

Most of those headers were actually added by express.js and/or fly.io for debugging and caching purposes.

But notice that nothing in the original request is stateful: it does not tell who or if someone is logged in to your site.*

Well actually, etags (a caching mechanism) can and have been used to track users.

So this is where HTTP cookies come in.

A server can tell the browser to include data in future requests to that website.
Let's say you wanted to know the last time someone visited your site (without saving that data in your own database).

Your express code could look like this:
```
app.get('/', (req, res) => {
  res.send('Hello World!');

  // sets a cookie called "visited" to the current time (in milliseconds)
  res.cookie('visited', Date.now().toString());
});
```

Now your response request will look like this:
```
200
Content-Type: text/html; charset=utf-8
Set-Cookie: visited=1664532412664
...More Headers...


... the webpage content ...
```

Future reqeuests will now send that data as one of the headers:

```
GET /
Cookie: visited=1664532412664
...More Headers...

```

Note that a website can set and read multiple cookies.
Also, cookies have associated settings (for instance, they could be set to expire, or set so that the js has no access to them).

You may have heard of tracking cookies, that comes from website's adding code from data-collection agencies.
Those codebytes will send a http request from the website to their servers with information about the site they came from.


Also note that several jurisdictions have set very strict laws for what cookies can be used for and what consent must be gathered before using them.
If you ever found yourself in Europe in recent
 years, you'd have noticed almost every site will pop up a big box saying "we value your privacy, please press 'accept' to let us track you."
That's because of the General Data Privacy Regulation (GDPR) that they passed.
You'll almost definitely end up working GDPR into your professional future architecture's design.

One of the most important thing about cookies is that they are linked to the _domain_.
That is, websites do not share cookies.
This is an important privacy and security consideration.
What's more, trying to access one website from another results in a Cross-Origin-Resource-Sharing (CORS) handshake, which requires an explicit opt-in flow by the other website's servers.


# Theoretical User Flow

Let's say that you want to support users on your website.

In theory, for each new user that clicks "register", you make a database entry to store info about them and allocate a unique id.

Then you would save that unique id as a cookie in the browser.

The next time they visit the page, the request will come with that cookie, and you can tell who it is and serve them content accordingly.

However, there are several issues with this design.

First, what if they want to login on a second computer.
In that case, you will need to authenticate them beyond just their computer.
Most commonly, that's solved with a username and password.
So you'd have to implement that, which is not as straightforward as you may think.
Passwords in general are insecure, it's much better to use your phone's face-id or fingerprint authentication.
In any case, you have to store the password (or more accurately, a salted-hash of the password) and that would then be prone to hacks and eventual user-data leaks.

Another issue is when the user clears their browser history.
In that case, they'd have to login again, and so username-password becomes a useful tool.

However, what if a user forgets their password?
Now you need a backup email to send a password-reset request through.
Now our initial registration is starting to get complicated.

And the worst thing of all, what about fake accounts?
We don't want to leave our website open to thousands of account creations from a single bot.

Instead, it's much safer (and easier for your users) to use a third party authentication service.

For this class, we'll implement github authentication.

# OAuth (Authorizing with a different service)

There is an internet standing for authenticating with different services called OAuth (really, OAuth 2).

OAuth stands for "Open Authorization"

It's a very standard protocol, so I thought I'd share a video for y'all.

https://www.youtube.com/watch?v=t18YB3xDfXI



