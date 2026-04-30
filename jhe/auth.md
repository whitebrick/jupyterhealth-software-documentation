---
title: Auth
---

## Authentication & Authorization

The OAuth 2.0 Authorization Code flow with PKCE is used to issue access and refresh tokens, while OIDC is used to issue ID tokens, for both Practitioners (via web login) and Patients (via JHE clients through a secure invitation link).

Endpoints and configuration details can be discovered from the OIDC metadata endpoint:
`/o/.well-known/openid-configuration`

Separate OAuth clients are created for the Web UI (Practitioners) and for individual JHE (Patient) Clients.

### JHE Client Auth Flow

> A JHE Client is the application component that communicates directly with JHE to manage consents and upload data.

In an effort to utilize the standard libraries ([Django OAuth Toolkit](https://github.com/django-oauth/django-oauth-toolkit)) without too much modification, the JHE Client Authorization Code grant flow is divided into three parts:
1. The first part of the flow used to generate the authorization code is instigated by the JHE server with no interaction from the client.
1. The authorization code is then shared with the client out-of-band (E-mail or SMS) as a secret invitation link.
1. The JHE Client at a later date then swaps the authorization code for an access token.

Because the Patient authorization code is instigated by the server (Step 1 above) rather than a web browser, the PKCE code challenge and code verifier must be static values and travel with the authorization code. The Patient JHE Client then sends this `code_verifier` along with the `authorization_code` to obtain tokens. The `redirect_uri` serves no purpose (as the initial authorization code has already been issued) but is required per OAuth spec so is defined as a static path (see below).

Bringing this altogether, the JHE Client requires 4 values to the invitation code used in the link
1. `host_with_port` - The host name (and optionally `:port`) of the JHE to connect with, eg example.com
2. `client_id` - The OAuth 2.0 Client ID
3. `authorization_code` - The OAuth 2.0 authorization code
4. `code_verifier` - The OAuth 2.0 code verifier

```
# EXAMPLE LINK
https://app.tcp.org/invitation/jhe.tcp.org~G7lkfTooTemBHzfya2wpGOZZSIbbtPH6joiVZvCF~psJUkrTx9ZJ2KHyG5Iz9F2NzrKdCL3~7sMHvAWzSEKIj2tSifAIFTruBmfLYriljxVBI5NyrQ

invitation_code = "jhe.tcp.org~G7lkfTooTemBHzfya2wpGOZZSIbbtPH6joiVZvCF~psJUkrTx9ZJ2KHyG5Iz9F2NzrKdCL3~7sMHvAWzSEKIj2tSifAIFTruBmfLYriljxVBI5NyrQ"

host_with_port, client_id, authorization_code, code_verifier = invitation_code.split("~")

post_url = parse(f"https://{host_with_port}/o/.well-known/openid-configuration").token_endpoint
redirect_url = f"https://{host_with_port}/auth/callback" # Doesn't do anything but required by spec

POST https://jhe.tcp.org/o/token/
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code&redirect_uri=https%3A%2F%2Fjhe.tcp.org%2Fauth%2Fcallback&client_id=G7lkfTooTemBHzfya2wpGOZZSIbbtPH6joiVZvCF&code=psJUkrTx9ZJ2KHyG5Iz9F2NzrKdCL3&code_verifier=7sMHvAWzSEKIj2tSifAIFTruBmfLYriljxVBI5NyrQ

RESPONSE
{
  "access_token":"1CxKTwNOK5vr0gnO6LY0ZfJF70prfy",
  "expires_in": 1209600,
  "token_type": "Bearer",
  "scope": "openid",
  "refresh_token": "DkZRFWWHB9Qndbv1WQn4UVOvKZbOz",
  "id_token": "eyJ0e..."
}
```

The returned `access_token` should be included in the `Authorization` header for all subsequent API requests with the prefix `Bearer `

> [!NOTE]
> It is understood using static values for PKCE defeats the intended purpose but because the JHE Client authorization code generation is instigated by the server and shared out of band (rather than a web browser) dynamic PKCE can not be used.

### Single Sign-On (SSO) with SAML2

The [django-saml2-auth](https://github.com/grafana/django-saml2-auth) library is included to support SSO with SAML2.

#### Example SAML2 Flow with [mocksaml.com](mocksaml.com)

Ensure you have the following System Settings

| Key | Value Type | Value | Notes |
|-----|-----------|-------|-------|
| `auth.sso.saml2` | int | `1` | Set to 0 to disable SAML SSO |
| `auth.sso.idp_metadata_url` | string | `https://mocksaml.com/api/saml/metadata` | SAML IdP metadata URL |
| `auth.sso.valid_domains` | string | `example.com,example.org` | Comma-separated email domains for SSO |


##### Test Flow

1. Visit https://mocksaml.com/saml/login and enter the fields below:

   - ACS URL: `http://localhost:8000/sso/acs/` (or substitute your hostname) **End with a trailing slash**

   - Audience: `http://localhost:8000/sso/acs/`

2. Enter any email name `@example.com`
2. Enter any password
2. Click Sign in
2. The JHE portal should be displayed with the user in the matching user name in the bottom left hand corner
