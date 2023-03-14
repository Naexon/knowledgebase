# JSON Web Tokes

An introduction can be found on the [official JWT website](https://jwt.io/introduction).

## General
* open standard defined in the [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519)
* allows secure transmission of information as a JSON object
* verify the integrity of the contained claims by digitally signing
    * using a secret with the HMAC algo
    * or public/private key pair (RSA or ECDSA)
* hide contained claims from other parties via encryption

## Usage
* Authorization
    * once a user is signed in, a JWT is used to access services, etc.
* Information Exchange
    * secure transmission of information
    * headers are used to verify that the content was not tampered with
    * signed tokes are protected against tampering but readable by everyone
        * don't put secrets in the payload or header unless the token is encrypted

## Structure
* typically looks like this `hhhhh.ppppp.sssss`
* Header
    * token type, which is JWT
    * signing algorithm 
* Playoad
    * contains claims
        * typically a statement about an entity (typically, the user)
        * registered claims
            * see [RFC 7519 4.1](https://www.rfc-editor.org/rfc/rfc7519#section-4.1)
        * public claims
            * defined at will by thos using JWT
            * must be collision free
                * via proper namespacing
                * registered in the [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)
        * private claims
            * predefined claims that the participating parties agree on
    * Base64Url encoded
* Signature
    * combination of the encoded header, payload, and a secret
