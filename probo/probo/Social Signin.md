Social Signin delegates the signin to a **OAuth2 Service** provided by a 3rd party such as google.   This has many benefits not least of which is the vast resources that these providers have to ensure that the service is secure.

On successfully signing in with an Oauth2 Service, the site can request details of the user from the service and use this to find an existing [[User]], or create a new [[User]].

