All of these APIs were chosen essentially by googling what REST APIs are out there. So these are all APIs that the developers describe as REST APIs. 

However, Fielding’s dissertation describes something fairly specific. He imagined a system where the server can scale up to practically the entire internet. That meant keeping the server clean of specific session data. If we imagine the trivia site with its session token (designed to prevent repeating questions) the way to make that truly RESTful is to have the client send up (in the request) all the questions it has already seen. That’s more expensive in terms of bytes sent over the network, but it saves the server from having to remember anything per client.

Authorization also often requires a session on the server.

A small comment that code-on-demand means that the shipped client does not need to know or implement everything the server might want it to do. Instead, the server can ship abilities down to the client with the request’s response. This is somewhat different than having the code open-sourced.

One thing that mattered to Fielding was that the server responses fully described themselves. When a resource comes over, it not only describes itself, but what can be done from there. That’s the core of the Hypermedia as the Engine of Application State. That would allow the system to be fully explorable by a crawler, and would make it so that a server could potentially expand or contract the API later, and the clients would figure it out without needing to be recompiled. I think technically, none of the APIs satisfy this (although I’m not certain about elastic search).

Ultimately, though, it doesn’t actually matter in the practical world whether an API satisfies all of Fielding’s constraints. Looking at the example requests you all shared, REST APIs seem to be, at their core, an HTTP client-server interaction where each resource is at its own url and each request is almost fully independent (except for authentication).

My advice for you in the professional world, whenever you use an API, you’ll have to read the docs. You can’t assume that they will actually satisfy Fielding’s constraints.

And when you design your own APIs, use Fielding’s constraints as guidelines if you want the benefits they give you.



