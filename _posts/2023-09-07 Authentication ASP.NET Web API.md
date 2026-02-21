# Authentication with ASP.NET Core WEB API

* Visual Studio "Create new project"-> "ASP.NET Core Web API" template, "No advanced features(https...)"
* [JWT Authentication And Authorization In .NET 6.0 With Identity Framework](https://www.c-sharpcorner.com/article/jwt-authentication-and-authorization-in-net-6-0-with-identity-framework/)
* [Authentication And Authorization In ASP.NET 5 With JWT And Swagger](https://www.c-sharpcorner.com/article/authentication-and-authorization-in-asp-net-5-with-jwt-and-swagger/)
* [JWT Authentication And Authorization In .NET 6.0 With Identity Framework](https://www.c-sharpcorner.com/article/jwt-authentication-and-authorization-in-net-6-0-with-identity-framework/)

## Why do we need Refresh Tokens?

Refresh tokens are the kind of tokens that can be used to get new access tokens. When the access tokens expire, we can use refresh tokens to get a new access token from the authentication controller. The lifetime of a refresh token is usually much longer compared to the lifetime of an access token.

We will set a short lifetime for an access token. So that, even the access token used by a hacker gets access only for a brief period. We will issue a refresh token along with an access token from the login request. Whenever the access token expires, we can get a new access token using the refresh token.
