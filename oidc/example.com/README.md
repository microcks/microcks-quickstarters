# Mock OIDC provider

The [OIDC specification](https://openid.net/developers/specs/) doesn't specify the paths of the HTTP endpoints that an Identity Provider must implement, and thus it cannot be described as an OpenAPI schema.

However, since OIDC relies on an endpoint discovery mechanism, and thanks to Microcks templating, we can use an OpenAPI schema as artifacts to get a partial mock of the OIDC protocol.

## Usage

The OIDC mocking is only partial.

Here are the current capabilities of the mock:

### Discovery (/.well-known/jwks.json)

Simply configure your OIDC client with `auth-server-url` pointing to your Microcks server with the path `/rest/OIDC/1.0`.
It will then automatically target the same server for all other OIDC endpoints.

### Asymmetric key signatures (/.well-known/jwks.json)

The OIDC client will trust any JWT signed with the [known private key](./privateKey.pem) matching the public key returned by the mock.
This can be used to generate JWT tokens with expected claims, and which will never expire, for instance using https://jwt.io/.

### Authorization (/authorize)

The Authorize endpoint is used to implement SSO from other WebUIs.
We can for instance use it from a Swagger UI, to get a Bearer access token to test our APIs.

In the example metadata, we answer an HTTP 302 Redirect to the provided redirect_uri, with a token_type=Bearer and an access_token value which is based on:

* the client_id parameter, to select a pre-built token (alice, bob, admin)
* or default to using the value of client_id as the access token itself.

Tokens need to be signed using the [private key](./privateKey.jwk), and can be generated using https://jwt.io/, with:

* "iss" claim set to "microcks", i.e. matching the issue field returned by the discovery endpoint
* "aud" claim set to the client_id of the service the token will be used to authenticate with
* "exp" claim set far in the future, to avoid token expiration issues
* Any other claim that the service expects, such as "sub", "scopes", etc.

### Token introspection (/tokeninfo)

If the token is opaque, the OIDC client needs to validate it via the introspection endpoint.
To simplify Microcks dispatch rules, we actually use non-opaque tokens that match the userID.

Custom scopes can be returned in the mock response, in case the application also relies on them for authorization.

### Token exchange (/token)

When the application needs to call another application, it needs to exchange the access token for a new one with the required scopes.
But when the other application is mocked, it likely doesn't care whether the token is actually valid!
Thus, the /token endpoint may return a dummy 'opaque' token.

## Credits

This mock is based on work or ideas from several projects:

* Gravitee.io [partial OIDC Swagger 2 specification](https://github.com/gravitee-io/gravitee-docs/blob/3836cb37ad3c4794cbe6df78cab4960321128961/am/2.x/oidc/swagger.yml)
* Another partial OAuth2 spec: https://github.com/fed239/oauth2-swagger/blob/master/oauth2-authorization_code.yaml
* [Quarkus](https://github.com/quarkusio/quarkus) `OIDCWiremockTestResource` behavior, and its JWKS key pair.
* [Keycloak](https://www.keycloak.org/), in particular for alice/bob/admin RBAC examples.





