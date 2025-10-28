# Authentication

```{note} This documentation is heavily based on [ftw.tokenauth](https://pypi.org/project/ftw.tokenauth/) product's documentation because
we are using this product under-the-hood to provide authentication services for API users.

```

CLMS Website Authentication is handled using [EU Login](https://ecas.ec.europa.eu/cas/help.html), European Commission's user authentication service. This means that no user password is stored in the CLMS Website, but the portal uses [Openid-connect](<https://en.wikipedia.org/wiki/OpenID#OpenID_Connect_(OIDC)>) to handle the user authorization with EU Login. By using EU Login the user only has to create one Europe-wide account and will be able to access all European services that use EU Login without needing to create a new account for each service.

That is why all users wanting to use the API need to create specific API tokens to communicate with the API.

Here we will explain all the steps needed to create and use such tokens.

The use of API authentication flow involves four steps:

1. A logged-in service user issues a service key in Plone and stores the private key in a safe location accessible to the client application.
2. The client application uses the private key to create and sign a JWT authorization grant.
3. The client application exchanges the JWT authorization grant for a short-lived access token at the @@oauth2-token endpoint.
4. The client then uses this access token to authenticate requests to protected resources.

Assuming the client is in possession of a service key, the flow looks like this:

```{image} ./images/authentication-flow.png
    :alt: Register/Login link
```

## Login in the CLMS Website

Using the `Register/Login` button in the top green bar of the portal, the user will be redirected to EU Login where he will be able to login or register a new account.

```{image} ./images/authentication-register-login-link.png
    :alt: Register/Login link
```

When the user authorization is finished and the user is redirected back to the CLMS Website, the name of the user will be shown in the top
green bar and the user will be able to access his profile page and create the needed API tokens.

```{image} ./images/authentication-register-login-user-name.png
    :alt: Register/Login with user name
```

## Create API Tokens

Clicking on the name in the top green bar, the user will be redirected to his profile page where he can fill the profile form and can also access some other options reserved for logged-in users.

Using the left menu present in the profile page the user can access the `API Tokens` section.

```{image} ./images/authentication-tokens-access.png
    :alt: Access API Tokens section.
```

In this section the user will see all the available API Tokens, that can be revoked or where a new one can be created.

```{image} ./images/authentication-tokens-page.png
    :alt: Access API Tokens section.
```

When clicking the `Create new Token` button a new form will be shown where the user can fill the name of the token:

```{image} ./images/authentication-tokens-create-new-token.png
    :alt: Access API Tokens section.
```

After filling the form the token will be created immediately and the token details will be presented in the form.

```{image} ./images/authentication-tokens-create-new-token-created.png
    :alt: Access API Tokens section.
```

It is very important to note that token details will be shown just once and just in this moment. The user will need to follow the details
and copy the token details to a file, in order to use the token in the future.

A service key looks like this (this specific token is revoked and no actions can be performed with it):

```{literalinclude} ./others/token.json
   :language: json
```

After creating one or more tokens the user will be able to delete them using the buttons in the token list view.

```{image} ./images/authentication-token-token-list.png
    :alt: Access API Tokens section.
```

After deleting the token you can create a new one. 

## Use the service token to sign the JWT authorization grant

In order to request an access token the client application will use the private service key created in the previous step to create and sign a JWT.

The JWT needs to contain the following claims:

| Name | Description                                                              |
| ---- | ------------------------------------------------------------------------ |
| iss  | Issuer - must be `client_id` from service key                            |
| aud  | Audience - must be `token_uri` from service key                          |
| sub  | Subject - must be `user_id` from service key or an arbitrary userid of   |
|      | an existing user if the service key user is allowed to impersonate other |
|      | users.                                                                   |
| iat  | The time the assertion was issued, specified as seconds since            |
|      | 00:00:00 UTC, January 1, 1970.                                           |
| exp  | The expiration time of the assertion, specified as seconds since         |
|      | 00:00:00 UTC, January 1, 1970. This value has a maximum of 1 hour after  |
|      | the issued time.                                                         |

The following is an example of how to build such a JWT token using python, provided the service key
is stored in a file called `my_saved_key.json`:

```python
import json
import jwt
import time

# Load saved key from filesystem
service_key = json.load(open('my_saved_key.json', 'rb'))

private_key = service_key['private_key'].encode('utf-8')

claim_set = {
    "iss": service_key['client_id'],
    "sub": service_key['user_id'],
    "aud": service_key['token_uri'],
    "iat": int(time.time()),
    "exp": int(time.time() + (60 * 60)),
}
grant = jwt.encode(claim_set, private_key, algorithm='RS256')

```

## Use the authorization grant to obtain the token

The client then makes a token request to the `token_uri` with the JWT grant it created.

This request needs to be a `POST` request with `Content-Type: application/x-www-form-urlencoded` and a request body that contains the form encoded parameters.

Two parameters are required:

| Name       | Description                                                    |
| ---------- | -------------------------------------------------------------- |
| grant_type | Must always be **urn:ietf:params:oauth:grant-type:jwt-bearer** |
| assertion  | The JWT authorization grant                                    |

```{http:example} curl wget python-requests
    :request: ./http-examples/authentication-access-token-request.req
```

The response will be in `application/json` format and will contain the access token needed to interact with the API.

```{literalinclude} ./http-examples/authentication-access-token-request.resp
   :language: http
```

## Use the access token to authenticate requests.

The client can then use the access token to authenticate requests. The token needs to be sent in the HTTP `Authorization` header as a `Bearer` token, as follows:

```{http:example} curl wget python-requests
    :request: ./http-examples/authentication-use-access-token.req
```

## Token expiration and renewal

Once the token expires the client must create a JWT authorization grant again and request a new access token.

The client should, instead of trying to predict access token expiration, just anticipate the case that authentication using an existing token will fail (because the token expired) and then perform the necessary steps to obtain a new token.

To accomplish this it is recommended to delegate all the requests a client application wants to make to a class that expects an `Access token expired` response as described above and obtains a new token if necessary. The failed request that leads to the error response then needs to be re-dispatched with its original parameters but with the new token in the `Authorization` header.

Care needs to be taken to not include an expired token (or any `Authorization` header for that matter) with the requests to the token endpoint when obtaining a new token.
