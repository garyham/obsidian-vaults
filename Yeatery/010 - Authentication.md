## Users Context

There are 3 UI interfaces into the Users Context.

-   AuthEmail - Allows users to register/link and sign_in using email/password credentials. Also allows them to change or reset their password.
    
-   AuthOAuth2 - Allows users to register/link and sign_in using a social provider (e.g. Google).
    
-   Users - Allows users to view their current User and AuthXXX information. Also allows System Users to create new Users (with roles) and associated AuthEmails.
    

Anyone can register as a user using either an email/password or social provider credentials. Registration will issue a session cookie.

Additionally, system users can be created by other system users without issuing a session cookie.

A user can have one or more authentication mechanisms (email, oauth2, active-directory) and can associate multiple authentication mechanisms with a single user (linking). A user must have a least one authentication mechanism.

All of the system users will have a corresponding User created with no authentication mechanism. This is to allow them to assume an identity and perform operations on the public side.

UserAuthEmailAuthOAuthhashas

#### User
```typescript
interface User {  
 id: uuid;  
 friendly_name: string;  
 roles?: string[]; // These are system roles and only assigned to system users.  
}
```

#### AuthEmail

```typescript
interface AuthEmail {  
 id: uuid;  
 email: string;  
 password_hash: string;  
 salt: string;  
 user_id: uuid (fk);  
}  
```


// email is unique constraint

#### AuthOAuth2

```typescript
interface AuthOAuth2 {  
 id: uuid;  
 provider_name: string;  // google etc.  
 provider_id: string;  
 provider_email: string;  
 photo_url?: string;  
 user_id: uuid (fk);  
}  
​  
// provider_name & provider_id are unique constraint
```


### API

#### OAuth2 - auth

The request must originate from an <a> tag so that the redirect will work correctly.

Method

URL

Operation

GET {session?}

/auth/oauth2/google

builds the redirect URL to the social provider. The callback includes state {session}, callback url, scopes etc for provider.

GET

/auth/oauth2/google/callback?code&state

Callback URL from the provider. Uses code to get token, then token to get user profile (different for each provider). The profile is normalised and used to either sign_in, if auth_oauth2 exists, or if session exists, link new auth_oauth2 to existing user, or if not create a new user and new auth_oauth2.

#### OAuth2 - rest

Method

URL

Operation

GET {session}

/user/oauth2

Return all the auth_oauth2s for this user.

GET {session}

/user/oauth2/{:id}

Get a single auth_oauth2.

DEL

/user/oauth2/{:id}

Remove an auth_oauth2. Only is it is not the last.

#### Email - auth

Method

URL

Operation

GET {session?}

/auth/email/sign_in?email&password

attempt to sign_in using credentials. On success, issue a cookie and redirect (client side) to private pages.

GET

/auth/email/verify?action=register|reset&email

client_side: redirect to code page. server_side: check if auth_email already exists. If exists, send notification email to email. If not exists, generate code, associate key (code) with value (email, action) and send code to email.

GET {session?}

/auth/email/verified?code&password

Check if new password meets strictness criteria. Check if code exists (could have expired or be wrong). Retrieve email & action using code. If action is reset, change password for auth_email(email, password). If action is register create a new user (if no session detected) and new auth_email. If a session exists, link a new auth_email to the session user. On success, issue a cookie and redirect (client side) to private pages.

#### Email - rest

Method

URL

Operation

GET {session}

/user/email

Return all the auth_emails for this user.

GET {session}

/user/email/{:id}

Get a single auth_email.

GET {session}

/user/email/{:id}/change_password?password&new_password

Change the password for an existing auth_email. Check if old_password is correct before changing. Check if new password meets strictness criteria.

DEL

/user/email/{:id}

Remove an auth_email. Only is it is not the last.

### Schema/Structs

Struct

Use

%User{}

%AuthEmail{}

%AuthOAuth2{}

### Commands

Commands are exposed by the Business Logic to be called by the API Layer.

#### Email

command

parameters

notes

returns

create_user_by_auth

%{email, password}

creates User and AuthEmail in single transaction.

%User{}

create_user_by_auth

%{provider_name, provider_id, friendly_name?, photo_url?}

creates User and AuthOauth2 in single transaction.

%User{}

create_user

%{friendly_name}

creates User

%User{}

create_auth

%{email, password}

creates an AuthEmail

%AuthEmail{}

create_auth

%{provider_name, provider_id, friendly_name?, photo_url?}

creates an AuthOAuth2

%AuthOAuth2{}

add_auth_to_user

%User{}, %AuthEmail{}

%User{}

add_auth_to_user

%User{}, %AuthOAuth2{}

%User{}

find_user_by_auth

%AuthEmail{}

Find %User{} where id = AuthEmail.user_id

%User{}

find_user_by_auth

%AuthOAuth2{}

Find %User{} where id = AuthOAuth2.user_id

%User{}

find_auth_email

Find all AuthEmail

%AuthEmail{} []

find_auth_oauth2

Find all AuthOAuth2

%AuthOAuth2{}[]

find_auth_email

email

Find AuthEmail by email address.

%AuthEmail{}

find_auth_oauth

provider_name, provider_id

Find AuthOAuth2 by provider name and id

%AuthOAuth2{}

set_password

%AuthEmail{}, password

Validates, then saves the password associated with this auth_email.

%AuthEmail{}

[[https://dev.solita.fi/2018/11/07/securing-websocket-endpoints.html]]