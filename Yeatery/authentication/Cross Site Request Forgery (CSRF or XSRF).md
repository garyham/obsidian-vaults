CSRF is an attack that relies on the authentication credentials (tokens or session ids) being stored in cookies, and therefore automatically sent when the user accesses that site.

The user is lured to page on evil.com (possibly by a phishing email).   Simply loading the page may be sufficient to trigger a request (using javascript onload())  to 
> https:://good.com/transfer?from=me&to=drevil&amount=100000

## Prevention
Don't use cookies to authenticate, instead copy the contents of the cookie to a header using JS. 
 


