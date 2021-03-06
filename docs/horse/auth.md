# Authentication

The authentication is based on [JWT](https://jwt.io/) (JSON Web Tokens), [Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) and and [HTTP Authentication Framework](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication).


## Process

Once a login request is accepted by the server, a `HttpOnly` cookie `jwt` and a normal cookie `csrf` are set.

When the client needs to access a resource, there are two authentication modes. The reasons are explained in the [Security](#security) section.

### Cookie Authorization

Add a CSRF header in the request header for `POST`, `PUT`, `PATCH` and `DELETE` requests:

`X-CSRF-Token: <csrf>`

If the CSRF header is not added, access is restricted to `GET` requests.

### HTTP Authentication

Get the JWT with an API:
 
`GET /api/v1/misc/jwt`

Add a authorization bearer with the JWT in the request header:

`Authorization: Bearer <jwt>`

You can use any HTTP method with HTTP Authentication. You do not need to add the CSRF header because HTTP Authentication doesn't suffer from CSRF attacks.

Note that you should not save the JWT anywhere (local storage, session storage, global variables, DOM element) to prevent XSS attacks.


## Why JWT?

> [Introduction of JWT](https://jwt.io/introduction)
> 
> JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed.

Another common used authorization method in web application is session. In a session setup, the client (browser) saves a session id (mostly in cookies), and the server saves a map of session id to user information in the database (e.g. redis), or in memory (e.e. memcache). The first method needs an extra table in the database, while the later method can not preserve the data after a server restart.

However, JWT is only saved on client-side and verified on server-side, which means the server doesn't need to save any information about logged-in users. This is called *STATELESS*. What's more, since the server only need to do a verification, no I/O (reading from the database or memory) is involved, the process will be typically faster than session authorization.

There are three parts in JWT, header, payload and signature. Each part is in JSON format and is transformed into base64 form when transmitting.

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
> For example:

```json
{
  "sub": "5f3cef33c4e40624828cadbf",
  "iat": 1614167646,
  "nbf": 1614167646,
  "jti": "6011ca7b-a7e3-44f3-b61e-81960ded8d72",
  "exp": 1615377246,
  "type": "access",
  "fresh": false,
  "name": "liuyh615",
  "scope": "sjtu",
  "channel": "jaccount"
}
```

You can check the meaning of some official claims in [RFC7519](https://tools.ietf.org/html/rfc7519#section-4.1).

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
+ [cross-site scripting (XSS) attacks](https://portswigger.net/web-security/cross-site-scripting): the attacker uses malicious JavaScript and sends sensitive data (such as session id or JWT) to another server. 
+ [cross-site request forgery (CSRF) attacks](https://portswigger.net/web-security/csrf): the attacker causes the victim user to carry out an action unintentionally.

We use these techniques to enhance the security of the website:

+ Do not save any sensitive data or token in Local Storage and Session Storage. Use cookies instead.
+ Use the `Secure` attribute so that a cookie is sent to the server only with an encrypted request over the HTTPS protocol, never with unsecured HTTP (except on localhost)
+ Use the `HttpOnly` attribute to prevent access to cookie values via JavaScript.
+ Use the `SameSite` attribute and a CSRF token to prevent CSRF attacks

Local Storage and Session Storage are insecure because any JavaScript can access the contents of these storages and send them to a remote server. Although we can prevent the injection of malicious code to our website, the client users are very likely to have a bunch of Chrome/Firefox extensions installed which can also access these storages. Similarly, a cookie without the `HttpOnly` attribute is also open to JavaScript.

> This isn't the full story. However now we can keep our cookies from being stolen via XSS attacks, but session cookies vulnerable to CSRF attacks.

We add a random CSRF Token in the JWT, and also set a cookie containing the CSRF Token without the `HttpOnly` attribute so that the client can retrieve it by JavaScript. When the client makes a `POST`, `PUT`, `PATCH` or `DELETE` request, an extra `X-CSRF-Token` header must be added to request headers. The server can compare the CSRF token in the JWT (via cookie) and that in the header to verify the client. 

Now the XSS attackers only know the CSRF token, and the CSRF attackers don't know the CSRF token, so both of them can not perform an attack.

However, no system is safe. If an attacker can perform XSS and CSRF attacks at the same time, the CSRF attack will be successful.

Check this [reference](https://indominusbyte.github.io/fastapi-jwt-auth/usage/jwt-in-cookies/) for details.



## Third-party Login Support

- [JAccount](http://developer.sjtu.edu.cn/wiki/JAccount)


