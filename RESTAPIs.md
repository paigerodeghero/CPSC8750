# Identifying REST APIs 

## Is it RESTful? Group Exercise
**Summary:** The goal of this exercise is to determine if your group's assigned APIs are RESTful. 

1. Go to your assigned API
2. Skim the APIs docs
3. Find and example of request and response
4. Determine if it complies with the constraints of REST (if the API requires authentication, note that and determine if the constraints are met given the authentication)
5. Create a PowerPoint presentation with the below template: 
	  
	  a. Slide 1: What API were you assigned and what does it do?
	  
	  b. Slide 2: Example request and response
	  
	  c. Slide 3: Which constraints it satisfies
	  
	  d. Slide 4. Is it RESTful?
6. Save PowerPoint to "Files" in the class's "General" channel 


## Reminder from last lecture
**The Constraints of REST:**
1. Client-server
2. Stateless server
3. Cache
4. Uniform interface

	a. Identification of resources
	
	b. Manipulation of resources through representations
	
	c. Self-descriptive messages
	
	d. Hypermedia as the engine of application state
5. Layered System
6. Code-on-demand (optional)

Video from the previous lecture: https://www.youtube.com/watch?v=6UXc71O7htc&t=126s&ab_channel=Apigee  

## Clarifications of Constraints
1. Client-Server: This is true if the client implementation and server implementation are totally independent of each other.
2. Stateless Server: The server should not have any locally maintained state in order to serve a request. This means that any sort of active session would violate the constraint. Even authentication would create a stateful response by the server.
3. Cache: Resources should come with a description of how long / if they can be cached
4. Uniform Interface
	a. Identification of Resources: All resources should be independently named and accessible through a URI
	b. Manipulation of resources through representations: All actions that can be done on a resource are uniquely represented by the URI and the HTTP verb (GET, POST)
	c. Self-descriptive message: A message should be fully described, so that any layer within the system can understand it and modify it appropriately
	d. Hypermedia as the engine of application state: The response that the API gives back should not only contain information about the current state, but information as to what can be done from that state (eg. the twitter webpage not only contains what content is there, but provides specific links for how to add content). If an API cannot be "crawled," then it does not satisfy this constraint
5. Layered System: The Client-Server interaction and Uniform Interface are sufficient to add in opaque layers. Things like a corporate firewall would be able to understand the request, or the server could utilize a CDN (which is a caching proxy that is closer to individual users than your primary servers).
6. Code-on-demand (optional): Whether the client can be handed code that expands its abilities and understanding of the API.

## API Group Assignments:
1. open trivia database (referenced in tutorial): https://www.sitepoint.com/rest-api/
^^ for this one, also explain what the tutorial defined as RESTful

2. tableau: https://help.tableau.com/current/api/rest_api/en-us/REST/rest_api.htm

3. jira: https://docs.atlassian.com/jira-software/REST/7.3.1/

4. salesforce: https://trailhead.salesforce.com/content/learn/modules/api_basics/api_basics_rest
^^ You can press "skip for now" when the tutorial asks you to sign in

5. home assistant: https://developers.home-assistant.io/docs/api/rest/

6. elastic: https://www.elastic.co/guide/en/elasticsearch/reference/current/api-conventions.html

7. shopify: https://shopify.dev/api/admin-rest

8. redmine: https://www.redmine.org/projects/redmine/wiki/Rest_api

9. woo: https://woocommerce.github.io/woocommerce-rest-api-docs/#introduction

10. skyscanner: https://developers.skyscanner.net/docs/intro

11. twitter api: https://developer.twitter.com/en/docs/twitter-api
