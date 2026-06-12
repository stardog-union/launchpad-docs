# Auth0 Configuration

Auth0 can be used as a login provider to authenticate users into Launchpad. Auth0 does not currently support SSO connections to Stardog endpoints.

## Login Provider Configuration

The following configuration options are available for Auth0 SSO.

> [!NOTE]
> See [How to Create an Auth0 Application to login with Auth0 in Launchpad](#how-to-create-an-auth0-application-to-login-with-auth0-in-launchpad) for additional information.

### `AUTH0_AUTH_ENABLED`

The `AUTH0_AUTH_ENABLED` setting is used to enable or disable Auth0 authentication to log users into Launchpad.

- **Required:** Yes (if using Auth0)
- **Default:** `false`

### `AUTH0_DOMAIN`

The `AUTH0_DOMAIN` is the domain of the Auth0 application used to log users into Launchpad (e.g. `your-tenant.us.auth0.com`).

- **Required:** Yes (if using Auth0)
- **Default:** not set

### `AUTH0_CLIENT_ID`

The `AUTH0_CLIENT_ID` is the client id of the Auth0 application used to log users into Launchpad.

- **Required:** Yes (if using Auth0)
- **Default:** not set

### `AUTH0_CLIENT_SECRET`

The `AUTH0_CLIENT_SECRET` is the client secret of the Auth0 application used to log users into Launchpad.

- **Required:** Yes (if using Auth0)
- **Default:** not set

### How to Create an Auth0 Application to login with Auth0 in Launchpad

#### 1. Create a new application in Auth0

- Sign into your [Auth0 Dashboard](https://manage.auth0.com)
- Navigate to **Applications** > **Applications**
- Click **"Create Application"**
- Name your application (e.g. "Stardog Launchpad")
- Choose **"Regular Web Applications"** as the application type
- Click **"Create"**

#### 2. Configure the application

- Open the application's **Settings** tab
- **Allowed Callback URLs**: `{BASE_URL}/oauth/auth0/redirect`
  - See [`BASE_URL`](../README.md#base_url) for more information on what the value should be.
- **Allowed Logout URLs**: Set this to `{BASE_URL}` if you want users to be redirected to the Launchpad home page after logging out.
  - See [`BASE_URL`](../README.md#base_url) for more information on what the value should be.
- Click **"Save Changes"**

#### 3. Note your application credentials

On the application's **Settings** tab, under **Basic Information**, make note of the following — you will need them when configuring Launchpad:

- **Domain**
- **Client ID**
- **Client Secret**

#### 4. Configure Launchpad

```bash
AUTH0_AUTH_ENABLED=true
AUTH0_DOMAIN=<auth0_domain>
AUTH0_CLIENT_ID=<client_id>
AUTH0_CLIENT_SECRET=<client_secret>
```

> [!NOTE]
> Launchpad requests the `openid`, `profile`, `email`, and `groups` scopes from Auth0 and identifies users by their `email`. Ensure the users you want to log in have an email address on their Auth0 profile.
