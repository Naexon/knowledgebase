# OAuth 2.0

The YouTube video [OAuth 2.0 and OpenID Connect (in plain English)](https://www.youtube.com/watch?v=996OiexHze0) explains everything very well.

## General
* downsides of a home-grown solutions
    * security
    * maintenance
* new usecases in around 2007
    * SSO
        * SAML was used and is still in use
            * very confusing spec and hard to work with as a developer
    * mobile app login
    * delegated authorization
        * this is what OAuth solves
        * give an application permissions to do somethings on your behave

## Terminology
* __Resource Owner__
    * owner of the data
    * e.g. the user
* __Client__
    * refers to the application that wants the data
* __Authorization Server__
    * system used to accept to give the information
* __Resource Server__
    * system that holds the data the client wants to get
* __Authorization Grant__
    * thing that proves that the resource owner allows the client to access the data
* __Redirect URI__
    * sometimes also called __Callback__
    * shows the authorization server, where the user ends up after completing the flow
* __Access Token__
    * key the client uses to get access to the data on the resource server
* __Scope__
    * any type of permissions that make sense in the system
    * the authorization grant has a list of scopes
        * e.g. read contacts, delete email, read email
* __Consent__
    * resource owner must give explicit consent to give access to some data
    * requested scopes are used by the auth server to generate the consent screen
* __Back Channel__
    * highly secure
    * normally on the backend
* __Front Channel__
    * less secure
    * normally in the browser


## OAuth Flow
* set of steps that result in the client to have access to some data of the resource owner
* steps
    1. (front channel) client requests some data from the user (data owner)
    2. (front channel) user clicks the corresponding button and gets redirected to the authorization server
        * includes the redirect URI
        * includes the type of response that is requested
            * commonly an authorization code
        * includes the scopes
    3. (front channel) authorization server requires the user to log in and consent to a level of permission
    4. (front channel) auth server redirects to the places specified in the redirect URI
        * includes the response
    5. (back channel) client takes the authorizaion code and exchanges for the access token
        * access token is scoped to the scopes the user consented to
    6. (back channel) client goes to the resource server and can access the data with the access token


Break at 33:00 min