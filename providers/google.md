# Google Configuration

> [!IMPORTANT]
> **Stardog Launchpad documentation has moved to [docs.stardog.com](https://docs.stardog.com/launchpad/).**
>
> This repository is an archive. It documents **Launchpad v3** and **Voicebox Service `0.x`**, and is no longer updated.
>
> For **Launchpad v4** and **Voicebox Service `v1.0.0`** or later, see:
>
> - [Stardog Launchpad](https://docs.stardog.com/launchpad/) — installing, configuring, and running Launchpad, and its login providers
> - [Voicebox Service](https://docs.stardog.com/launchpad/voicebox-service/) — running, configuring, and deploying the Voicebox Service
> - [Launchpad release notes](https://docs.stardog.com/release-notes/stardog-cloud/stardog-launchpad/)
> - [Voicebox Service release notes](https://docs.stardog.com/release-notes/stardog-voicebox/)

![Google](../assets/google-logo.png)

Google can be used as a login provider to authenticate users into Launchpad. Google does not currently support SSO connections to Stardog endpoints.

## Login Provider Configuration

The following configuration options are available for Google SSO.

> [!NOTE]
> See [How to Create a Google OAuth2.0 Client to login with Google in Launchpad](#how-to-create-a-google-oauth-20-client-to-login-with-google-in-launchpad) for additional information.

### `GOOGLE_AUTH_ENABLED`

The `GOOGLE_AUTH_ENABLED` is used to enable or disable Google authentication to log users into Launchpad.

- **Required:** Yes (if using Google)
- **Default:** `false`

### `GOOGLE_CLIENT_ID`

The `GOOGLE_CLIENT_ID` is the client id of the Google OAuth2.0 client used to log users into Launchpad.

- **Required:** Yes (if using Google)
- **Default:** not set

### `GOOGLE_CLIENT_SECRET`

The `GOOGLE_CLIENT_SECRET` is the client secret of the Google OAuth2.0 client used to log users into Launchpad.

- **Required:** Yes (if using Google)
- **Default:** not set

### How To Create a Google OAuth 2.0 Client to login with Google in Launchpad

1. **Create OAuth 2.0 Client**
  - Go to [Google Cloud Console](https://console.cloud.google.com)
  - Create a new project or select existing project
  - Navigate to **"APIs & Services"** > **"Credentials"** in the left-hand vertical menu
  - Click **"Create Credentials"** > **"OAuth client ID"**
  - Choose **"Web application"** type
  - Name your OAuth 2.0 client (e.g. "Stardog Launchpad")

2. **Configure Authorized Redirects**
  - *( Add redirect URI )*: `{BASE_URL}/oauth/google/redirect`
   - See [`BASE_URL`](../README.md#base_url) for more information on what the value should.

3. **Generate Credentials**
  - Click **"Create"**
  - Google will display *Client ID* and *Client Secret*
  - Copy these values immediately or download the JSON file containing the credentials.

4. **Configure Launchpad Environment Variables**

```bash
GOOGLE_AUTH_ENABLED=true
GOOGLE_CLIENT_ID=<client_id>
GOOGLE_CLIENT_SECRET=<client_secret>
```