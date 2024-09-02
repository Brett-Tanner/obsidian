LINK includes APIs, authentication, non-UI data delivery in bulk, and web applications for clients to use financial data.  
  
Will sometimes be more backend or frontend heavy, typically on a project/product for a couple months and the nature of that project decides what type of work you're doing.  
  
Interview process is:  
1. Phone screen  
2. Technical Interview  
3. Career deep dive  
  
May also be a <8 hour take home, but he doesn't think so for this role.  
  
#### OAuth  
- For authorisation, not authentication  
- Allows an app to perform certain actions on resource owner behalf  
-  Client requesting permission redirects RO to the auth server, which asks if RO wants to grant that client permission to take certain actions.
- Resource server is what the client wants permission to access, can be different from the auth server
- Redirect/callback URL is where the auth server redirects the RO back to after authorisation. Can have different response types, the most common is a code.
	- Client ID identifies the client, client secret is shared by client and auth server to privately share info
- The code/auth grants a 'scope', a set of granular actions the client has access to
- Auth server sends client a temp auth code, which client sends back with client id/secret to obtain an access token used with the resource server
  
#### OpenID  
- Built on OAuth, concerned with authentication  
- Auth servers supporting this are 'Identity Providers', as they can create a login session with your identity attached rather than authorising a client to take actions on your behalf
- Initial request process is same as Oauth, but requests OPENID scope
- Receives an ID token (JWT) in addition to the access token, which unlike the access token is decodable by the client

### Sessions 
- User authenticates, server creates a session & stores in session store (could be DB, redis etc.). 
- Then sends back a cookie with session ID, which is sent with each request & used to verify the session against the session store
- Easy to revoke, but in distributed environment add complexity as all servers need access to same session store

### JWTs
- Authenticate, create & sign JWT with secret key. 
- No separate storage needed, rather than session being stored on backend all necessary data is in the JWT
- Can be difficult to invalidate
- HMAC is simpler as same private key used to sign & verify, but RSA etc. better as only public key needs to be shared. However adds computation overhead
- HMAC fine if a monolith or only sharing JWTs with own services, otherwise public/private key
- Best to expire JWTs quickly so they're less useful if stolen, and provide a longer lasting refresh token which can be checked for validity against an auth server much like a session ID would

### Message Queues
- Like a channel in Go, store events to be dispatched to workers
- Need some way of being persisted if the server goes down
- Can be grouped into transactions, which can be rolled back or committed

### Load Balancing


### Caching


### Webapps ~> native  
- WebView allows you to embed a browser in a native app, use your existing site and give it access to native APIs through JS  
  
## Questions  
  
- Is there on-call? How often is it/used?  
- Seems like there's a lot involved in creating/maintaining banking API integrations, at least a month to develop a new API integration. Plus APIs can change, need to be re-done. How much time do you spend on that?  
- Typical workload/type for a sprint?
- React/Rails versions

### Take-home

- Design considerations
	- SQLite because keeping it all in memory is fragile
	- Assuming the DB exists in the state described in the instructions

- Need a cart which stores the scanned items as a hash.
	- Maybe fetches & stores prices per item at init so less lookups
		- Or, fetch them as needed because could be millions of items and store once retrieved
	- Should be able to enter quantity after scanning, or just hit enter for 1
	- Only update price for scanned item & total cart, otherwise could be slow for huge carts
	- Should print the cart after scanning each item
		- Calculate col width by finding the longest value in that col, then adding some padding constant to it
	- Create new cart

- Need a list of discounts, should be able to add your own
	- Item condition, represented by a hash with the number of items needed to trigger the discount
	- A price condition, which must be exceeded for the discount to apply
	- An 'exclusive' boolean, which prevents other discounts being applied. 
		- Exclusive discounts should be checked first, and if one applies stop looking at others
		- Maybe a priority for exclusive discounts? As multiple could apply
	- Think about what new 'types' of discounts are likely, how they could be added
		- Maybe the just need to all be in a 'conditions' column keyed by the type of conditions, which is used to call the relevant method if it exists

- Line items
	- Will need the item, its barcode and the price

- Tests
	- System test for adding some items which trigger a discount and seeing the correct output on the command line
	- Adding a line item to the cart
	- Calculating without discounts
	- Printing the cart to the terminal
		- Need to figure out how to test CLI output
		- [Here](https://stackoverflow.com/questions/11349270/test-output-to-command-line-with-rspec)
	- Calculating with a discount

- Improvements
	- Checking the DB exists and is in the correct state
	- Get the product/discount data from an API
	- Add 'discount groups', which are exclusive apart from the base one
	- Persist old carts, allow retrieval