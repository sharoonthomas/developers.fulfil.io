Authentication
==============

Before you can interact with the Fulfil API, your app must provide the 
necessary authentication credentials in each HTTP request that you
make to Fulfil. The way to provide these credentials depends on the 
type of app that you're developing. 


Fulfil API offers two forms of authentication:

API Key
-------

Also know as the V1 authentication, this allows users to create an
API key associated to their user account to build private applications.

All activity performed using this API key will reflect as though the
user performed the action.

Use this for personal bots that need to do something with Fulfil on
behalf of you.

On the other hand, if you are building an app that is likely to be used
by other users in your company or publicly by other Fulfil merchants, you
should use the OAuth2 authentication system instead.


OAuth2
------

OAuth2 an open protocol to allow secure authorization in a simple and 
standard method from web, mobile and desktop applications.

An app built with OAuth2 can interact with the Fulfil API on behalf of
multiple users (within an organization) or on behalf of multiple Fulfil
merchants.

:doc:`Implement OAuth2 </getting-started/oauth2>`