---
title: Authentication and Authorization for SSO
date: 2023-12-20 23:54:06
categories:
- Authentication, Authorization
tags:
- OAuth 2.0
- OpenID
- JWT
- SSO
---

## OAuth 2.0 (Open Authorization 2.0):
  OAuth 2.0 is an <strong>authorization framework</strong> that allows third-party applications to obtain limited access to a user's resources on a server without exposing their credentials. It's commonly used for delegated authorization scenarios, such as allowing a third-party application to access a user's data on another service (e.g., social media data access).
  If the Client is a Single-Page App (SPA), an application running in a browser using a scripting language like JavaScript, there are two grant options:
  - Authorization code (front channel + back channel): safer since the Access Token is not exposed on the client side, and this flow can return Refresh Tokens.
  - Implicit (front channel only)
<!--more-->
  [Which OAuth 2.0 Flow Should I Use?](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use)
  [OAuth 2.0 and OpenID Connect (in plain English)](https://www.youtube.com/watch?v=996OiexHze0)
  [Is it secure to store a refresh token in the database?](https://stackoverflow.com/questions/59511628/is-it-secure-to-store-a-refresh-token-in-the-database-to-issue-new-access-toke)

## OpenID Connect (OIDC):
  OpenID Connect is an <strong>interoperable authentication protocol based on the OAuth 2.0 framework</strong> . It provides a standardized way for clients to obtain information about the end-user in order to authenticate them. OIDC is often used for single sign-on (SSO) scenarios, where a user can log in once and access multiple applications without needing to log in again. OIDC uses JSON Web Tokens (JWT), HTTP flows and avoids sharing user credentials with services.

  What OpenID Connect adds
  - ID token(JWT)
  - UserInfo endpoint for getting more user information
  - Standard set of scopes
  - Standardized implementation

  ```sh
    ------------------
    | OpenID Connect |  -> OpenID Connect is for authentication
    ------------------
    |    OAuth 2.0   |  -> OAuth 2.0 is for authorization
    ------------------
    |      HTTP      |
    ------------------
  ```
  [How OpenID Connect Works - OpenID Foundation](https://openid.net/developers/how-connect-works/)
## JSON Web Tokens (JWT):
  JSON Web Tokens (JWT) are <strong>a format for representing claims</strong>, often used in OAuth 2.0 and OIDC as a means of transmitting information securely between parties. In OIDC, the identity token is a JWT.

  JWTs consist of three parts:

  - Header: typically consists of two parts: the type of the token, which is JWT, and the signing algorithm being used, such as HMAC SHA256 or RSA.
    ```
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```
  - PlayLoad(claims): there are three types of claims: registered([reference](https://datatracker.ietf.org/doc/html/rfc7519#page-10)), public([reference](https://www.iana.org/assignments/jwt/jwt.xhtml)), and private(custom) claims.
  - Signature: used to verify the message wasn't changed along the way
    ```
    HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
    ```

  The output is three Base64-URL strings separated by dots.
  ```sh
  Base64[Header].Base64[Payload(claims)].Base64[Signature]
  ```

  usage
  ```
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
  ```

[Introduction to JSON Web Tokens](https://jwt.io/introduction)