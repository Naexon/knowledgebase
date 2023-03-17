# OpenID Connect

The YouTube video [OAuth 2.0 and OpenID Connect (in plain English)](https://www.youtube.com/watch?v=996OiexHze0) explains everything very well.

## General
* around 2009
    * login with Facebook
    * login with Google
* OAuth meant for Authorization but not Authentication
    * Authentication implemented with OAuth and some custom hacks
* OpenID Connect as an addon for OAuth
    * meant for __Authorizaion__
    * adds an __ID Token__ which holds user information

## OpenID Connect Flow
* very similar to the OAuth flow
* in the initial request, we also request an __openid scope__
* in the code exchange, we also get an ID token back
    * encoded as a JWT
* sometimes with the access token we can also request more info from the __userinfo endpoint__
* "we are using OpenID Connect to do OAuth"
    * only difference is the request scope
    
