```Registration``` is the process of creating an account for a user.  This may be done by the user themselves, or by an administrator.  With OAuth2 it is not possible to distinguish between registering and logging in, both follow the same flow.

[[Authentication]] is the process by which we confirm that a user is who they say they are.  This can be done using an email/password combination, an email confirmed by clicking a link (a.k.a. passwordless login) or using a 3rd party OAuth2 provider such as Google or using an application such as Authy for [[Multi Factor Authentication]].

Authentication establishes a **Session** for a single **user**.


## User
A user can have one or more methods of signing in.

## Email/Password Signin
An email and password combination is required to sign in.  This combination is usually set either by the user themselves (self registration) or by an administrator.  

## Passwordless Signin
A user supplies their email and a smart-link is sent to that email.  On clicking that link, the user will be signed in and redirected back to the website.  

> The email used may already have a AuthEmail and therefore a password.

## Social Signin
Using Google (or another social password vendor) to both register and signin.

## AuthEmail
Authenticating with an email is perhaps the most complex mechanism to support because of the possible use cases.

### Registering
If allowed, registering by email is a 2 stage process.  The user first enters their email address.  This will cause a email with a code to be sent to that address.  This code is used to confirm the email belongs to the user.  Once confirmed a user is created (if not already signed in) and an new AuthEmail will be created.

We want to use the AuthEmail as the means to log in with, or without, a password i.e. support passwordless authentication.

With passwordless authentication, the user begins the sign-in (or register) process by entering their email.  A link is then sent to the email which, when clicked, will complete the sign-in (or registration) process and issue a session cookie.   The link is a one-time only use and will expire in 5 minutes if not used.

This means that some AuthEmail records may exist without and salt/hash.

How do we set a password for a user who registered using passwordless?  If t****hey register, the existance of an AuthEmail record for that email usually means that this email is already taken, however in this case it is possible for record to exist, but have no associated password.  

Registering with a password automatically enables passwordless sign-in.

Does this mean we should allow a user to change a password if none exists.



| field      | notes                                                                                                 |
| --------- | ----------------------------------------------------------------------------------------------------- |
| email     | string up to 128 characters.                                                                          |
| salt_hash | string.  It is a combined salt and hash because that is the way some algorithms encrypt the password. |
| user_id   | FK to the the user                                                                                    |
| verified  | **not required** - the AuthEmail can only be created after the email has been used to confirm the registration                                                                                                       |
 
 > There is no need for an is_verified column because the AuthEmail can only be created after the email has been used to confirm the registration.

```typescript
interface {
	email: string; // primary, unique
	user_id: string;
	salt_hash: string;
	profile: integer // 16 bit should be more than ample.
}
```

| 1st col                                             | 2nd col                                                              |
|:--------------------------------------------------- |:-------------------------------------------------------------------- |
| This is a test of the 1st column to check on sizing | and the 2nd which is in theor and the 2nd which is in theory smaller |