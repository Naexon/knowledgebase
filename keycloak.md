# Keycloak

In this file, I will write down any knowledge I find on Keycloak and deem important for me in the future. It is not comprehensive in any way and just for me to remember stuff.

In the [Keycloak Documentation](https://www.keycloak.org/documentation), the guides __Server Administrator__ and __Server Developer__ are interesting to me. The prior goes over the general concepts and the Admin processes in configuring Keycloak through the UI. The latter includes the Docs and examples for implementing custom code.

## Service Provider Interface (SPI)

* adding custom code to Keycloak is done over SPIs
* we realize them by creating a service config file and implementing a corresponding factory
    * e.g.: Theme Selector SPI
        * provide a class that implements `ThemeSelectorProvider`
        *  the corresponding factory implementing `ThemeSelectorProviderFactory`
        * adding the factory implementation to the config file `META-INF/services/org.keycloak.theme.ThemeSelectorProviderFactory

## Authentication SPI

* Keycloak already includes different auth mechanisms
    * e.g.: Kerberos
        * computer-network auth protocol for providing mutual authentication
        * uses symmetric-key crypto
        * needs trusted third party
        * maybe uses public-key crypto for certain parts of the auth process
    * e.g.: passwords
    * e.g.: One-Time-Passwords (OTPs)
        * password only valid for one login session or transaction
        * often incorporates two-factor auth
        * combines _something a person has_ (e.g.: a keyring fob device or specific mobile phone) and _something a person knows_ (e.g.: a PIN)
        * often use randomness to seed shared keys that are then hashed
* we use the Auth SPI to implement custom mechanisms

### Keywords
* __Auth Flow__
    * container for all authentications and actions that must happen during a login or registration
    * specifically, it contains executions (that bind the flow to the above mentioned)
    * can contain subflows
* __Authenticators__
    * holds the logic for an authentication or action
    * usually a singleton
* __Executions__
    * binds the authenticator to the flow
    * binds the flow to the configuration of the authenticator
* __Execution Requirement__
    * defines how a execution behaves in a flow
    * layering done by using subflows
    * different types
        * disabled
            * does not count to mark a flow as successful
        * required
            * must be successfully sequentially executed
            * flow terminates if one of these executions fails
        * conditional
            * only set on subflows
            * if all executions succeed the conditional subflow acts like a required one
            * if any execution fails it acts like a disabled one
        * alternative
            * only one of the executions must succeed for the flow to succeed
            * will not execute if a required execution is also present
        * (enabled)
* __Authenticator Config__
    * configuration for an Authenticator for specific execution environment
    * each execution can have a different configuration
* __Required Action__
    * one-time actions that need to be completed before a user is logged in after a successful authentication
    * e.g.: reset an expired password

### Auth Algorithm
* overview
    1. run all executions in a flow
    2. if the flow is successful (see Execution Requirements) associate the user with the current session
    3. check for required actions and render the pages if necessary
    4. if the required actions have been resolved, the user is logged in
* detailed steps
    1. first the OpenID Connect or SAML protocol provider unpack all relevant data and verify the client and any signatures
    2. a `AuthenticationSessionModel` is set up which looks up the flow and starts it
    3. if a execution should be run, the corresponding provider is loaded
    4. some executions require a user that is already associated with the auth sessions
        * i.e. a `UserSessionModel` is verified and associated with the `AuthenticationSessionModel`
        * also, if a user is required and not provided the execution fails
    5. a providers return a status
        * examples
            * `success()` (in most executions)
                * the execution succeeded
                * in an alternative execution, all following alternative execution will not run
                * in a required execution, the flow fails
            * `attempted()` (e.g. in token evaluations)
                * not an error but not a success either
            * `challenge()` (e.g. in forms)
                * in an alternative execution the challenge response is held until all other alternative executions are evaluated
                * in an required execution the challenge is always honored by the flow
                * depending on the response a `success()` or `attempted()` is returned
            * `forceChallenge()` (e.g. in Kerberos logins)
                * in an alternative execution the corresponding challenge cannot be ignored by the flow and must be handled before continuing to the next execution
            * `failureChallenge()` (e.g. in username/password forms)
                * challenge that is logged as an error
    5. after the flow is completed the auth processor associates a `UserSessionModel` with the `AuthenticationSessionModel`
    6. the auth processor checks if the state might trigger an required action
        * this is done by the `evaluateTriggers()` method
        * e.g.: the realm has a password expiration policy and a password is too old and needs to be reset
    7. for each required action the `requiredActionChallenge()` method is called which renders the corresponding page
    8. after all required actions have been resoved, the user is logged in

### Authenticator Interface
* in order to create an Authenticator, we must at least implement the `org.keycloak.authentication.AuthenticatorFactory` and `org.keycloak.authentication.Authenticator` interfaces
* Authenticators that need the user to input data that is validated against some database are called __CredentialValidators__
    * they also need a class that extends `org.keycloak.credential.CredentialModel`
    * and a class that implements `org.keycloak.credential.CredentialProvider`
    * plus the corresponding class implementing `org.keycloak.credential.CredentialProviderFactory`

### CredentialModel Class
* Credentials
    * are stored in the database in the Credentials table
    * stucture
        * _ID_
        * _user_ID_
        * _credential_type_
            * set during creation
            * must reference an existing credential type
        * _created_date_
        * _user_label_
        * _secret_data_
            * static json data that cannot be transmitted outside of Keycloak
            * should be salted or hashed in the database
                * requires additional fields for the salt, the algorithm and iterations
        * _credential_data_
            * static json that is shared in the admin console or the REST API
        * _priority_
            * defines which credential is preferred by the user in case multible credentials are present
* extend `CredentialModel` class
    * `public static final String TYPE` constant corresponds to the _credential_type_ we write in the database
    * create a `CredentialData` and a `SecretData` class
        * correspond to the _credential_data_ and _secret_data_ field respectivly
        * add a public constructor that can deserialize the JSONs
            * e.g. with the Jackson `@JsonCreator` and `@JsonProperty` annotations
        * add public getters
    * also store the classes in corresponding `final` variables
        * can be private
    * in order to work correctly we need to save both, the raw JSON data and the unmarshalled objects
        * we can read everything from a simple `CredentialModel` that we get from the database
        * see [here](https://www.keycloak.org/docs/latest/server_development/index.html#extending-the-credentialmodel-class) for a detailed walkthrough
* I will not iterate on the the implementation of a `CredentialProvider` as it is pretty explanatory when reading the corresponding docs [here](https://www.keycloak.org/docs/latest/server_development/index.html#implementing-a-credentialprovider)
    * same for `Authenticator` (see [here](https://www.keycloak.org/docs/latest/server_development/index.html#implementing-an-authenticator)), `AuthenticatorFactory` (see [here](https://www.keycloak.org/docs/latest/server_development/index.html#implementing-an-authenticatorfactory)), adding an auth form (see [here](https://www.keycloak.org/docs/latest/server_development/index.html#adding-an-authenticator-form)), and adding an Authenticator to a flow (see [here](https://www.keycloak.org/docs/latest/server_development/index.html#_adding_authenticator))