# Authentication

The authentication is based on [JWT](https://jwt.io/) (JSON Web Tokens)
, [Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) and
and [HTTP Authentication Framework](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)
.

[comment]: <> (## Process)

[comment]: <> (Once a login request is accepted by the server, a `HttpOnly` cookie `jwt` and a normal)

[comment]: <> (cookie `csrf` are set.)

[comment]: <> (When the client needs to access a resource, there are two authentication modes. The)

[comment]: <> (reasons are explained in the [Security]&#40;#security&#41; section.)

## Query Parameters

Most endpoints under `/auth` use the same set of query parameters:

+ cookie (boolean) - If set to true (by default), the server will add Set/Delete-Cookie
  on response header. This works on direct call to the api in the browser, and also
  works on most browsers in fetch/XHR requests.
+ response_type (string, required) - one of "redirect" and "json". If set to "redirect",
  the response will be an HTTP redirect response, which is used for direct call to the
  api in the browser; if set to "json", the response will be a JSON response, which is
  mostly used for fetch/XHR requests in clients.
+ redirect_url (string) - the redirect url used in redirect response if `response_type`
  is "redirect". The default value is the host name of the server. It has no effect on
  other `response_type`.

## Process

### OAuth2 Login

#### List OAuth2 Clients

You can use this api to get a list of supported OAuth2 clients on the server:

`GET /api/v1/oauth2`

#### Login

First, you should call this api to get a redirect url for the OAuth process:

`GET /api/v1/oauth2/<oauth_name>/authorize?response_type=redirect`

In most case, `response_type` should be set to "redirect" because OAuth requires a
redirection to a thrid-party server and redirects you back to the server. `cookie`
should be set to true because you must do so preserve the state during redirections. The
user will be redirected to the `redirect_url` given in the response.

#### Check Login Status

After the redirection, you can call the API:

`GET /api/v1/auth/token?response_type=json`

to check whether the login is success or whether the OAuth account is already linked to
a user. In the decoded JWT, there is a key `category`, which have two possible values: "
user" and "oauth".

If the value of `category` is empty, there should be some internal server error about
the oauth.

If the value of `category` is "user", you have already login. There is a
key `oauth_name` which indicates the OAuth service name you requested. The key `sub` is
the user id on the server. This is an example of the JWT payload of a successful login:

```json
{
  "sub": "0b88d9ad-4351-415b-a656-cdc30124b93c",
  "iat": 1635504162,
  "nbf": 1635504162,
  "jti": "ccab2e1d-4e28-443b-be0a-17952fd2e323",
  "exp": 1636713762,
  "type": "access",
  "fresh": false,
  "csrf": "2aaa82f2-f951-46fd-a09f-91006ad6d9d8",
  "category": "user",
  "username": "tc-imba",
  "email": "liuyh615@126.com",
  "student_id": "",
  "real_name": "",
  "role": "user",
  "oauth_name": "github"
}
```

If the value of `category` is "oauth", it means that we can't find a user related to the
OAuth account. The key `oauth_name` also indicates the OAuth service name you requested.
However, the other fields have different meanings. `sub` is the user id on the OAuth
service, and `username`, `email`, `student_id`, `real_name` are all obtained from the
OAuth service. You can display the information in the register page. See the register
section for details. This is an example of the JWT payload of an unsuccessful login
because of no user can be found by this OAuth account:

```json
{
  "sub": "006182C2-0B74-4FF6-9C1B-BEE57FF5076A",
  "iat": 1635506154,
  "nbf": 1635506154,
  "jti": "639fed64-dca3-4494-b1a0-3e4d9e15de68",
  "exp": 1636715754,
  "type": "access",
  "fresh": false,
  "csrf": "d50d1e9f-82dd-47b6-8e4a-ef53bc3fb5ca",
  "category": "oauth",
  "username": "liuyh615",
  "email": "liuyh615@sjtu.edu.cn",
  "student_id": "515370910207",
  "real_name": "刘逸灏",
  "role": null,
  "oauth_name": "jaccount"
}
```

### Password Login (Experimental)

If a user never set a password (this happens when they register via OAuth), he can not
use password login. Call the api to use password login:

`POST /api/v1/auth/login`

`response_type` can be either `redirect` or `json` according to the client design.

Use the same check of `category` after the login, though the `category` can only be "
user" because no OAuth process is used. The JWT payload is same as the one in OAuth
where  `category` is "user", except that `oauth_name` will always be an empty string.

### Register

The register api is

`POST /api/v1/auth/register`

It accepts a JSON form:

```json
{
  "username": "string",
  "email": "string",
  "password": "string",
  "oauth_name": "string",
  "oauth_account_id": "string"
}
```

`username` and `email` must be unique in the site, we will examine it.

`password` should be validated on client side (like input twice?). We haven't added any
restriction to the password on server side now.

#### Register by Username and Password

`oauth_name` and `oauth_account_id` should be set to empty string or null. An example of
the register form is:

```json
{
  "username": "liuyh615",
  "email": "liuyh615@126.com",
  "password": "drowssap"
}
```

#### Register by OAuth

`oauth_name` and `oauth_account_id` must be provided together and match exactly what is
defined in JWT if this is a register by OAuth. You should use `oauth_name` in JWT
as `oauth_name` in the form and `sub` in JWT as `oauth_account_id` in the form. You can
omit any of `username` and `email` so that the server will use the same values in JWT (
but it's possible that `username` and/or `email` is already registered and in this case
you should notify the user to bind the account instead of register a new one)
. `password` can also be omitted in a register by OAuth and the user may set it later
because the user can log in the account with OAuth now. A minimal example of the
register form is:

```json
{
  "oauth_name": "jaccount",
  "oauth_account_id": "006182C2-0B74-4FF6-9C1B-BEE57FF5076A"
}
```

You can also provide a full register form:

```json
{
  "username": "liuyh615",
  "email": "liuyh615@126.com",
  "password": "drowssap",
  "oauth_name": "jaccount",
  "oauth_account_id": "006182C2-0B74-4FF6-9C1B-BEE57FF5076A"
}
```

Note that the `email` provided by OAuth is `liuyh615@sjtu.edu.cn`. You can overwrite it
by providing `liuyh615@126.com` in this form.

### Logout (Experimental)

`POST /api/v1/auth/logout`

It will delete all related cookies if the `cookie` parameter is set to true. It doesn't
have any other side effect now, because JWT is stateless. So if you clean the cookies on
client side, you don't even need to call this api.

Maybe we will add a parameter on this api to log out the OAuth account for the user.
This is because if we don't log out the OAuth account, the OAuth login will always log
in the same user.

### Refresh and Fresh Token (Experimental)

`POST /api/v1/auth/refresh`

The JWT access token usually have a short expire time (several days), and the JWT
refresh token have a longer one (maybe one month). After calling this api, two new JWT
access and refresh tokens are generated and their expiry time are refreshed. However,
the key `fresh` in the JWT access token is set to false.

The key idea of JWT refresh token is that it help reduce the login frequency of users.
For example, we mark some api endpoints such as changing password and updating profile
to require a fresh token, and other api endpoints don't need it. The user must log in
again to get a fresh token for certain endpoints.

Now we don't define any api endpoint which requires a fresh token, but we will make some
of them later. (GitHub also has this feature, we can refer to it.)

### Link Account with OAuth (Not Implemented)

Already supported in the DB design. User will be able to link more than one OAuth
account (but one each OAuth service) and login the same account via any of them.

## Backends

Currently, there are two authentication backends implemented: cookie backend and jwt
backend.

### Cookie Authentication

#### Introduction

If the `cookie` parameter is set to true in the authentication, two `HttpOnly`
cookies `access_token` and `refresh_token` are set, you can not get their values through
JavaScript for security reasons, and any future requests (both direct and fetch/XHR)
will automatically contain the cookies.

If CSRF protect is enabled (in production mode), two extra cookies `csrf_access_token`
and `csrf_refresh_token` are set, you can (and need to) get their value through
JavaScript because they are not `HttpOnly` cookies.

#### Usage

For `GET` requests, you can call the APIs directly after the cookies are set.

For `POST`, `PUT`, `PATCH` and `DELETE` requests, you need to add a CSRF header in the
request header If CSRF protect is enabled:

`X-CSRF-Token: <csrf>`

If the CSRF header is not added, access is restricted to `GET` requests.

### HTTP Authentication

#### Introduction

If the `response_type` parameter is set to "json", you will get a json response which
contains two token strings `access_token` and `refresh_token`. Both tokens are encoded
in jwt format and contains basic information about the current user.

you should not save the JWT anywhere, especially in long term storages (local storage,
session storage); in short-term storages such as global variables and DOM element, it's
also not recommended storing them to prevent XSS attacks. It seems that some long term
storages listed above are secure, but browser plugins may still be able to steal them.
So the correct thing to do is each time when initialization the frontend (or client),
get the JWT with an API:

`GET /api/v1/auth/token?response_type=json`

You can get the tokens if the `HttpOnly` cookies are correctly set in the
authentication, and no CSRF header is needed because it's a `GET` request. If you get
empty tokens, it means that you need to go through the auth process to login first.

#### Usage

Add an authorization bearer with the JWT access token in the request header:

`Authorization: Bearer <access_token>`

You can use any HTTP method with HTTP Authentication. You do not need to add the CSRF
header because HTTP Authentication doesn't suffer from CSRF attacks.

## Why JWT?

> [Introduction of JWT](https://jwt.io/introduction)
>
> JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed.

Another common used authorization method in web application is session. In a session
setup, the client (browser) saves a session id (mostly in cookies), and the server saves
a map of session id to user information in the database (e.g. redis), or in memory (e.g.
memcache). The first method needs an extra table in the database, while the later method
can not preserve the data after a server restart.

However, JWT is only saved on client-side and verified on server-side, which means the
server doesn't need to save any information about logged-in users. This is called *
STATELESS*. What's more, since the server only need to do a verification, no I/O (
reading from the database or memory) is involved, the process will be typically faster
than session authorization.

There are three parts in JWT, header, payload and signature. Each part is in JSON format
and is transformed into base64 form when transmitting.

### Header

> The header typically consists of two parts: the type of the token, which is JWT, and the signing algorithm being used, such as HMAC SHA256 or RSA.
>
> For example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

> The second part of the token is the payload, which contains the claims. Claims are statements about an entity (typically, the user) and additional data.
>

Check the examples of JWT payload in [Process](#Process).

You can check the meaning of some official claims
in [RFC7519](https://tools.ietf.org/html/rfc7519#section-4.1).

### Signature

> To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.
>
> For example if you want to use the HMAC SHA256 algorithm, the signature will be created in the following way:

```javascript
HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
```

> The signature is used to verify the message wasn't changed along the way, and, in the case of tokens signed with a private key, it can also verify that the sender of the JWT is who it says it is.

## Security

There are mainly three kinds of attack on web applications:

+ man-in-the-middle (MitM) attacks: can be simply avoided with HTTPS protocol
+ [cross-site scripting (XSS) attacks](https://portswigger.net/web-security/cross-site-scripting):
  the attacker uses malicious JavaScript and sends sensitive data (such as session id or
  JWT) to another server.
+ [cross-site request forgery (CSRF) attacks](https://portswigger.net/web-security/csrf):
  the attacker causes the victim user to carry out an action unintentionally.

We use these techniques to enhance the security of the website:

+ Do not save any sensitive data or token in Local Storage and Session Storage. Use
  cookies instead.
+ Use the `Secure` attribute so that a cookie is sent to the server only with an
  encrypted request over the HTTPS protocol, never with unsecured HTTP (except on
  localhost)
+ Use the `HttpOnly` attribute to prevent access to cookie values via JavaScript.
+ Use the `SameSite` attribute and a CSRF token to prevent CSRF attacks

Local Storage and Session Storage are insecure because any JavaScript can access the
contents of these storages and send them to a remote server. Although we can prevent the
injection of malicious code to our website, the client users are very likely to have a
bunch of Chrome/Firefox extensions installed which can also access these storages.
Similarly, a cookie without the `HttpOnly` attribute is also open to JavaScript.

> This isn't the full story. However now we can keep our cookies from being stolen via XSS attacks, but session cookies vulnerable to CSRF attacks.

We add a random CSRF Token in the JWT, and also set a cookie containing the CSRF Token
without the `HttpOnly` attribute so that the client can retrieve it by JavaScript. When
the client makes a `POST`, `PUT`, `PATCH` or `DELETE` request, an extra `X-CSRF-Token`
header must be added to request headers. The server can compare the CSRF token in the
JWT (via cookie) and that in the header to verify the client.

Now the XSS attackers only know the CSRF token, and the CSRF attackers don't know the
CSRF token, so both of them can not perform an attack.

However, no system is safe. If an attacker can perform XSS and CSRF attacks at the same
time, the CSRF attack will be successful.

Check
this [reference](https://indominusbyte.github.io/fastapi-jwt-auth/usage/jwt-in-cookies/)
for details.

## Third-party Login Support

- [JAccount](http://developer.sjtu.edu.cn/wiki/JAccount)
- [GitHub](https://docs.github.com/en/developers/apps/building-oauth-apps/authorizing-oauth-apps)


