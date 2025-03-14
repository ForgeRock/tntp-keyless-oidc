# Keyless OIDC 

## Overview
This guide provides a step-by-step process to enable passwordless biometric authentication on Forgerock Identity Cloud using Keyless. Forgerock and Keyless have partnered to deliver a true passwordless authentication experience for both workforce and consumer applications.

Keyless will be set up as both an OpenID Connect (OIDC) service provider and an OpenID Connect identity provider (Social Identity Provider) for Forgerock Identity Cloud.

## Authentication: Configure Social Identity Provider
### Steps to Configure
1. **Log into Forgerock Identity Cloud Platform Admin Console** for your tenant.
2. From **Platform Admin Console Dashboard**, select the realm where configuration is needed.
3. Navigate to **Native Consoles → Access Management**.
4. Click on **Services** tile and select **Social Identity Provider Service**.
5. Click on **Service Management Tile** and select **Social Identity Provider Service**.
6. Under the **Secondary Configurations** tab, click **Add a Secondary Configuration** dropdown and select **OIDC Provider**.

### Client Configuration for OIDC Provider
Fill in the following details:

| Parameter | Description | Example |
|-----------|-------------|----------|
| Name | Select a name from Social IdP configuration | Keyless |
| Auth ID Key | OIDC claim that identifies the user | sub |
| Client ID | OIDC Client ID (Provided by Keyless) | - |
| Client Secret | OIDC Client Secret (Provided by Keyless) | - |
| Authentication Endpoint URL | OAuth authentication endpoint URL (Provided by Keyless) | - |
| Access Token Endpoint URL | OAuth access token endpoint URL (Provided by Keyless) | - |
| Well Known Endpoint | Well Known endpoint URL (Provided by Keyless) | - |
| Issuer | OIDC Issuer URL (Provided by Keyless) | `https://<my-keyless-tenant-fqdn>` |
| Client Authentication Method | Authentication method for OIDC Client | CLIENT_SECRET_POST |
| PKCE Method | OIDC PKCE configuration | S256 |
| Response Mode | OIDC Response mode | DEFAULT |
| Oauth Scopes | OIDC/OAuth scope parameter | openid profile email |
| Scope Delimiter | Scope delimiter | `<single-space-character>` |
| OIDC Endpoints | Authorization, token, userinfo, JWKS endpoints (provided by Keyless) | `https://<my-keyless-tenant>/connect/authorize` |
| Redirect URL | OIDC redirect from Keyless IDP upon completion of authentication | `https://<my-forgerock-tenant>/am/oauth2/realms/root/realms/<my-realm-name>/client/form_post/<Social-IDP-Name>` |
| UI Config Properties | buttonDisplayName | Keyless |
| UI Config Properties | buttonImage | `https://<my-keyless-tenant>/static.keyless.svg` |
| Transform Script | Script to normalize incoming claims from Keyless IDP | (See below) |

### Sample Normalization Script (Groovy)
```groovy
// Normalization script for Keyless Social IdP
import static org.forgerock.json.JsonValue.field
import static org.forgerock.json.JsonValue.json
import static org.forgerock.json.JsonValue.object

return json(object(
    field("mail", rawProfile.preferred_username),
    field("alias", selectedIdp + '-' + rawProfile.preferred_username.asString())
))
```

### Configure Authentication Tree
1. From **Realm Dashboard**, select **Authentication → Trees**.
2. Click **Create Tree**, provide a name (e.g., `KeylessAuth`).

#### Sample Authentication Tree
- **KeylessAuth**: Uses Keyless as the sole authentication mechanism.
- **Alternative Auth Tree**: Provides both password-based and Keyless authentication.

#### Accessing the Authentication Page
```url
https://<my-forgerock-tenant>/am/XUI/?realm=/<my-realm-name>&authIndexType=service&authIndexValue=<my-Auth-Tree-Name>#/
```

## Keyless Enrollment: OIDC SP/RP Configuration
### Steps to Configure
1. From **Realm Dashboard**, go to **Applications → OAuth 2.0 → Clients → Add Client**.
2. Provide required details:

| Parameter | Description |
|-----------|-------------|
| Client ID | e.g., `KeylessEnrollmentClient` |
| Client Secret | Generate a client secret |
| Redirect URIs | Provided by Keyless |
| Scope & Default Scope | `openid profile cn mail` |

3. Click **Create** and proceed with configuration.

### Advanced Tab Configuration
1. **Grant Types**: Select `Authorization_Code & Implicit`
2. **Token Endpoint Authentication Method**: Select `client_secret_post`
3. **Custom Properties**: `preferred_username=mail`

### OIDC Tab Configuration
1. **Client Session URI**: Provided by Keyless
2. **Post Logout Redirect URI**: Provided by Keyless
3. **Backchannel Logout URI**: Provided by Keyless
4. **Post Logout Redirect URI**: Based on realm name, e.g., `https://<forgerock-tenant>/enduser/?realm=<realm-name>#/dashboard`

5. Click **Save** to complete the configuration.

## Keyless Enrollment
1. Navigate to the **Keyless enrollment URL** (provided by Keyless).
2. Authenticate using your **Forgerock Identity Cloud credentials**.
3. Browser will redirect to **Keyless enrollment page**.
4. Download **Keyless authenticator app** on your mobile device.
5. Scan the **QR code** displayed on the Keyless enrollment page.
6. Enrollment completes successfully.

## Keyless Authentication
1. Navigate to an application secured via **Forgerock Identity Cloud SSO** (e.g., **Forgerock Identity Cloud end user dashboard**):
```url
https://<my-forgerock-tenant>/am/XUI/?realm=/<my-realm-name>&authIndexType=service&authIndexValue=<my-Auth-Tree-Name>#/
```
2. Click on **Continue with Keyless**.
3. Provide your **email registered with Keyless**.
4. Receive a notification on your **mobile device**.
5. Complete biometric authentication using **Keyless**.

---
This guide provides all necessary steps to integrate Keyless passwordless authentication with Forgerock Identity Cloud. For additional support, refer to the official documentation of Forgerock and Keyless.
