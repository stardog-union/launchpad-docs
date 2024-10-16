# Configuring Snowflake with Azure OAuth

This example documents the configuration of Stardog and Snowflake so that a user that was authenticated by Microsoft Entra ID (formerly Azure AD) using OAuth can access Snowflake from Stardog using their credentials "passed through" (rather than using a shared "system" account).

This configurations requires the registration of three applications in Microsoft Azure; one for Snowflake, one for the Stardog server, and one for Launchpad.

The first step is to configure Snowflake so that it can accept OAuth-authenticated JDBC connections from Stardog, which is the client application from Snowflake's perspective. Snowflake has instructions for this [here](https://docs.snowflake.com/user-guide/oauth-azure). This Snowflake guide supports two options:
1. The authorization server can grant the OAuth client an access token on behalf of the user.
2. The authorization server can grant the OAuth client an access token for the OAuth client itself.

We'll follow option 1 for passing through the Stardog user's credentials to Snowflake.

For the next two steps, we'll configure the Snowflake resource and the Snowflake client. The latter will serve as the app registration for the Stardog server.

## Configure the (Snowflake) OAuth resource in Microsoft Entra ID
Under **App Registrations**, **New registration**, we'll call this application `Example Snowflake Resource`, **Supported account types** is set to **Single Tenant**. Click **Register**.

Click **Expose an API**, click **Add a scope**. Accept the **Application ID URI** (e.g.: `api://08bcca47-1234-48ae-b92e-24f52263d55f`). This value will be referred to as the **<SNOWFLAKE_APPLICATION_ID_URI>** in subsequent configuration steps. Under **Add a scope**, provide `session:scope:<snowflake_role>` as the **Scope name**. In this example, the role will be `SYSADMIN` so we'll enter `session:scope:sysadmin`. Permit admins and users to consent. Provide admin and user display names and descriptions, e.g., `Admin SYSADMIN Role`; Grant the client to operate with the `SYSADMIN` role. Click **Add scope**. In this example, a scope will be added with name `api://08bcca47-1234-48ae-b92e-24f52263d55f/session:scope:sysadmin`.

## Create an OAuth client (Stardog Server) in Microsoft Entra ID.
Under **App Registrations**, **New registration**, well call this application `Example Stardog Resource`, **Supported account types** is set to **Single Tenant**. Click **Register**.

In the **Overview** section, copy the **ClientID** from the **Application (client) ID** field. This will be known as the **<OAUTH_CLIENT_ID>** in the following steps (e.g.: `ad6ad6ae-1234-46c2-8137-caac1396e658`).

Click on **Certificates & secrets** and then **New client secret**. (You can alternatively [use a certificate](https://docs.stardog.com/operating-stardog/security/oauth-integration#client-credentials-for-stardog-server).) Add a description of the secret. Click **Add**. Copy the secret. This will be known as the **<OAUTH_CLIENT_SECRET>** in the following steps (e.g.: `GcN8Q~8u4wAjMkduDog4T2eh4E6vQvwASRRXPdy~`).

Click on **API Permissions**, **Add permission**, **My APIs**. Select the `Example Snowflake Resource` that you created in [#create-an-oauth-client-stardog-server-in-microsoft-entra-id](*Configure the (Snowflake) OAuth resource in Microsoft Entra ID*) step. Click on the **Delegated Permissions** box. Check on the **Permission** related to the **Scopes** defined in the **Application** that you wish to grant to this client.

(Optional, and if enabled) Click on the **Grant Admin Consent** button to grant the permissions to the client. Click **Yes**.

Collect Azure AD information for Snowflake:
Click on **App Registrations**, **Example Snowflake Resource**, **Endpoints** in the Overview interface, note the following URLs:
* The **Authority URL** (Accounts in this organizational directory only) will be known as **<AZURE_AD_AUTHORITY_URL>**.
* The **OAuth 2.0 authorization endpoint (v2)** will be known as the **<AZURE_AD_OAUTH_AUTHORIZE_ENDPOINT>** in the following configuration steps. The endpoint should be similar to `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/oauth2/v2.0/authorize`.
* The **OAuth 2.0 token endpoint (v2)** will be known as the **<AZURE_AD_OAUTH_TOKEN_ENDPOINT>** in the following configuration steps. The endpoint should be similar to `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/oauth2/v2.0/token`.
* For the **OpenID Connect metadata**, open the link in a new browser window (e.g. `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/v2.0/.well-known/openid-configuration`). Locate the `jwks_uri` parameter and copy its value. It will be known as the **<AZURE_AD_JWS_KEY_ENDPOINT>** in the following configuration steps. The endpoint should be similar to `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/discovery/v2.0/keys`.
* For the **Federation metadata document**, open the URL in a new browser window (e.g. `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/federationmetadata/2007-06/federationmetadata.xml`). Locate the `entityID` parameter in the XML Root Element and copy its value. It will be known as the **<AZURE_AD_ISSUER>** in the following configuration steps. The entityID value should be similar to `https://sts.windows.net/ff24ca66-abcd-1234-8acf-43f2635ada42/`.

## Create a security integration in Snowflake (with audiences)

In a Snowflake SQL Worksheet as the `ACCOUNTADMIN` role, create a securuty integration with the following command.

```
create security integration external_oauth_azure_2
    type = external_oauth
    enabled = true
    external_oauth_type = azure
    external_oauth_issuer = '<AZURE_AD_ISSUER>' -- https://sts.windows.net/ff24ca66-abcd-1234-8acf-43f2635ada42/
    external_oauth_jws_keys_url = '<AZURE_AD_JWS_KEY_ENDPOINT>' -- https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/discovery/v2.0/keys
    external_oauth_audience_list = ('<SNOWFLAKE_APPLICATION_ID_URI>') -- api://3fa43612-dae5-4b09-9611-42f73b3cfa7a (defaults to <account_identifier>.snowflakecomputing.com, according to https://docs.snowflake.com/en/sql-reference/sql/alter-security-integration-oauth-external)
    external_oauth_token_user_mapping_claim = 'upn'
    external_oauth_snowflake_user_mapping_attribute = 'login_name';
```

For example:
```
create security integration example_external_oauth_azure
    type = external_oauth
    enabled = true
    external_oauth_type = azure
    external_oauth_issuer = 'https://sts.windows.net/ff24ca66-abcd-1234-8acf-43f2635ada42/'
    external_oauth_jws_keys_url = 'https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/discovery/v2.0/keys'
    external_oauth_audience_list = ('api://08bcca47-1234-48ae-b92e-24f52263d55f')
    external_oauth_token_user_mapping_claim = 'upn'
    external_oauth_snowflake_user_mapping_attribute = 'login_name';
```

If there is already an integration with this issuer, add the **<SNOWFLAKE_APPLICATION_ID_URI>** to the audience list:

```
alter security integration existing_external_oauth_azure set
    external_oauth_audience_list = ('<SNOWFLAKE_APPLICATION_ID_URI>', <existing_list_members>, ... );
```

For example:
```
alter security integration existing_external_oauth_azure set
    external_oauth_audience_list = ('api://08bcca47-1234-48ae-b92e-24f52263d55f', <existing_list_members>, ... );
```

If the scope is mapped to a new role (say, `VGUSER`, unlike our running example, where we are using `SYSADMIN`), create the ROLE and GRANT permissions to it. For example:
```
GRANT SELECT ON ALL TABLES IN SCHEMA VGDB.PUBLIC TO ROLE VGUSER;
GRANT USAGE ON SCHEMA VGDB.PUBLIC TO ROLE VGUSER;
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE VGUSER;
```

## Test connection between Stardog and Snowflake

As an interim step, we'll test that Stardog can authenticate with Snowflake over JDBC using OAuth. Choose a user for this test. For this example, we'll use `test.account@mycompany.com`. Ensure this user exists in Microsoft Entra ID with a password and in Snowflake, with **login_name** attribute value set to the **<AZURE_AD_USER_USERNAME>**, and with membership in the role associated with the Scope we registered earlier (`SYSADMIN`, in this example).

```
DESCRIBE USER "test.account@mycompany.com";
property		value
NAME			test.account@mycompany.com
DISPLAY_NAME		test.account@mycompany.com
LOGIN_NAME		TEST.ACCOUNT@MYCOMPANY.COM
PASSWORD		********
MUST_CHANGE_PASSWORD	FALSE
DISABLED		FALSE
DEFAULT_WAREHOUSE	COMPUTE_WH
DEFAULT_ROLE		SYSADMIN
...			...
```

Next, we'll perform a manual call to the **<AZURE_AD_OAUTH_AUTHORIZE_ENDPOINT>** to allow the user to authorize the Stardog client to access the Snowflake resource. This step may not be necessary if the administrator performed this authorization at an ealier step.

Take your Snowflake scope, which was `<SNOWFLAKE_APPLICATION_ID_URI>/session:scope:<snowflake_role>` (e.g. `api://08bcca47-1234-48ae-b92e-24f52263d55f/session:scope:sysadmin`). We'll call that **<FULLY_QUALIFIED_SCOPE>**. Now URL Encode the **<FULLY_QUALIFIED_SCOPE>** (you can use online resources like [https://www.urlencoder.org/] to do the encoding). We'll call this **<URL_ENCODED_QUALIFIED_SCOPE>**. For our example, the encoded URL is: `api%3A%2F%2F08bcca47-1234-48ae-b92e-24f52263d55f%2Fsession%3Ascope%3Asysadmin`

For **<URL_ENCODED_REDIRECT_URL>** we'll use: `http%3A%2F%2Flocalhost`
For **<STATE>**, any string will work. We'll use: `plugh`

The authorize URL will be: `<AZURE_AD_OAUTH_AUTHORIZE_ENDPOINT>?client_id=<OAUTH_CLIENT_ID>&response_type=code&redirect_uri=<URL_ENCODED_REDIRECT_URL>&response_mode=query&scope=<URL_ENCODED_QUALIFIED_SCOPE>&state=<STATE>`

For example: `https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/oauth2/v2.0/authorize?client_id=ad6ad6ae-1234-46c2-8137-caac1396e658&response_type=code&redirect_uri=http%3A%2F%2Flocalhost&response_mode=query&scope=api%3A%2F%2F08bcca47-1234-48ae-b92e-24f52263d55f%2Fsession%3Ascope%3Asysadmin&state=plugh`

Make sure your are logged in as the user you wish to have consent to the client app having access. If not, navigate to: [https://login.microsoftonline.com/organizations/oauth2/v2.0/logout]

Navigate to the authorization URL and login using the Microsoft credentials of the user we wish to grant consent.

Once logged in correctly, this should present two consent screens:
> test.account@mycompany.com  
> Permissions requested (1 of 2 apps)  
> Example Stardog Resource  
> App info  
> This application is not published by Microsoft.  
> This app would like to:  
> User SYSADMIN Role (Example Snowflake Resource)  
> View your basic profile  
> Maintain access to data you have given it access to  

> test.account@mycompany.com  
> Permissions requested (2 of 2 apps)  
> Example Snowflake Resource  
> mycompany.onmicrosoft.com  
> This application is not published by Microsoft.  
> This app would like to:  
> View users' basic profile  
> Maintain access to data you have given it access to  

This will result in this harmless error message. (We did this to grant consent. The error has to do with obtaining an authorization code, which we do not need. Alternatively you maybe redirected to a non-existant `localhost` URL if the client application was configured with `http://localhost` as a redirect URL. Regardless, the result can be ignored.)
> Sorry, but we’re having trouble signing you in.  
> AADSTS500113: No reply address is registered for the application.

We should now be able to obtain an access token permitting Stardog to access Snowflake using our user's credentials. We can do this with the following curl command:
```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" \
  --data-urlencode "client_id=<OAUTH_CLIENT_ID>" \
  --data-urlencode "client_secret=<OAUTH_CLIENT_SECRET>" \
  --data-urlencode "username=<AZURE_AD_USER>" \
  --data-urlencode "password=<AZURE_AD_USER_PASSWORD>" \
  --data-urlencode "grant_type=password" \
  --data-urlencode "scope=<FULLY_QUALIFIED_SCOPE>" \
  '<AZURE_AD_OAUTH_TOKEN_ENDPOINT>'
```

For our example:
```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded;charset=UTF-8" \
  --data-urlencode "client_id=ad6ad6ae-1234-46c2-8137-caac1396e658" \
  --data-urlencode "client_secret=GcN8Q~8u4wAjMkduDog4T2eh4E6vQvwASRRXPdy~" \
  --data-urlencode "username=test.account@mycompany.com" \
  --data-urlencode "password=xxxxxxxxxxx" \
  --data-urlencode "grant_type=password" \
  --data-urlencode "scope=api://08bcca47-1234-48ae-b92e-24f52263d55f/session:scope:sysadmin" \
  'https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/oauth2/v2.0/token'
```

This should return a JSON response like: `{"token_type":"Bearer","scope":"api://74b40656-67ab-4844-9068-1407a6d830ee/session:scope:sysadmin","expires_in":4910,"ext_expires_in":4910,"access_token":"eyJ0eXAi..."}`

You can decode this using online resources like <http://jwt.io>:
```
{
  "aud": "api://08bcca47-1234-48ae-b92e-24f52263d55f",
  "iss": "https://sts.windows.net/ff24ca66-abcd-1234-8acf-43f2635ada42/",
  "iat": 1728587575,
  "nbf": 1728587575,
  "exp": 1728592786,
  "acr": "1",
  "aio": "ATQAy/8YAAAA8QbK6YNJgDR5DIkI1fo8kbzYqFaVTO8RehezbKZHh0RLdhfa4L0iV0nnBTKUI+Ed",
  "amr": [
    "pwd"
  ],
  "appid": "ad6ad6ae-1234-46c2-8137-caac1396e658",
  "appidacr": "1",
  "family_name": "Account",
  "given_name": "Test",
  "ipaddr": "108.31.29.75",
  "name": "Test  Account",
  "oid": "c49289bc-be8f-47f3-b637-8240574cb847",
  "rh": "0.ARcAZsok_6q7702Kz0PyY1raQlYGtHSrZ0RIkGgUB6bYMO5oAYo.",
  "scp": "session:scope:sysadmin",
  "sub": "6hpsRRwuMaC1N-H3dGowpNfs9hjEsX2v6K1EW33uE4Y",
  "tid": "ff24ca66-abcd-1234-8acf-43f2635ada42",
  "unique_name": "test.account@mycompany.com",
  "upn": "test.account@mycompany.com",
  "uti": "TkCJXBLNQUiRxqodzcWOAQ",
  "ver": "1.0"
}
```

Note the `aud` claim matches the **<SNOWFLAKE_APPLICATION_ID_URI>**, `appid` matches **<OAUTH_CLIENT_ID>**, `scp` contains the scope we requested, and `upn` matches the user. The `upn` claim (**User Principal Name**) here should map to the `login_name` from: `DESCRIBE USER "test.account2@mycompany.onmicrosoft.com";`

This mapping from `upn` to `login_name` was set by the `external_oauth_token_user_mapping_claim = 'upn`' and `external_oauth_snowflake_user_mapping_attribute = 'login_name'` properties of the `create security integration` statement.

The token can be verified by Snowflake by running the following SQL command:
```
select system$verify_external_oauth_token('eyJ0eXAi...') Result;
```

This should return:
```
Token Validation finished.{"Validation Result":"Passed","Issuer":"https://sts.windows.net/ff24ca66-abcd-1234-8acf-43f2635ada42/","Extracted User claim(s) from token":"test.account@mycompany.com"}
```

This token can be used (until it expires) in the `ext.token` datasource property when setting up a Stardog datasource. An example property file looks like this:
```
jdbc.url=jdbc:snowflake://12345.us-east-1.snowflakecomputing.com/?db=VGDB
jdbc.driver=net.snowflake.client.jdbc.SnowflakeDriver
ext.authenticator=oauth
ext.token=eyJ0eXAi...
```

This properties file can be tested with the Stardog CLI in isolation (no Stardog server required), which can be downloaded from <https://downloads.stardog.com/stardog/stardog-latest.zip>. This requires copying the Snowflake JDBC driver to the `client/cli` folder of the Stardog install. The command works like this:
```
# List all schemas
$ stardog-admin virtual source_metadata snowflake_oauth.properties

# List all tables in <schema> 
$ stardog-admin virtual source_metadata snowflake_oauth.properties <schema>

# List all columns in table <schema>.<table>
$ stardog-admin virtual source_metadata snowflake_oauth.properties <schema> <table>
```

You can also create a virtual graph named `snowflake_oauth` (without passthrough authentication) at this stage. This mappings file presumes a table with the following DDL:
```
CREATE TABLE "Student_Sport"(
      "Student" varchar(50),
      "Sport" varchar(50)
);
INSERT INTO "Student_Sport" ("Student","Sport") VALUES ('Venus', 'Tennis');
INSERT INTO "Student_Sport" ("Student","Sport") VALUES ('SMITH', 'Hackeysack');
```

The mappings file:
```
MAPPING
FROM SQL {
  SELECT CURRENT_USER the_user, *
  FROM PUBLIC."Student_Sport"
}
TO {
  ?s <urn:sport> ?Sport .
  ?s <urn:asSeenBy> ?the_user .
}
WHERE {
  BIND(TEMPLATE("urn:person:{Student}") AS ?s)
}
```

Note the incorporation of a `CURRENT_USER` function that will become handy when we test passthrough later.
```
$ stardog query <DB name> 'select * from <virtual://snowflake_oauth> { ?s ?p ?o }'
+------------------+--------------+--------------------------------+
|        s         |      p       |               o                |
+------------------+--------------+--------------------------------+
| urn:person:Venus | urn:asSeenBy | "test.account@mycompany.com" |
| urn:person:Venus | urn:sport    | "Tennis"                       |
| urn:person:SMITH | urn:asSeenBy | "test.account@mycompany.com" |
| urn:person:SMITH | urn:sport    | "Hackeysack"                   |
+------------------+--------------+--------------------------------+

Query returned 4 results in 00:00:00.694
```

## Set up Stardog Client Application
For this example we'll use [Launchpad](https://docs.stardog.com/launchpad/) as the Stardog Client Application. These instructions are adapted from <https://github.com/stardog-union/launchpad-docs/blob/main/README.md> and <https://github.com/stardog-union/launchpad-docs/blob/main/azure/access-token-passthrough-mode.md> (a different kind of passthrough).

In the app registration for the Snowflake client (**Example Stardog Resource**), under **App roles**, create app roles for `app_reader` and `app_writer`, making them available to users and groups. These names should match the names of Stardog groups that you want to allow to athenticate to Stardog using OAuth. Users that are assigned to these app roles will be allowed to authenticate and will be granted membership to those Stardog groups.

In the Azure portal, go to **Enterprise Applications**, select the Snoflake Client application (`Example Stardog Resource`). Navigate to **Manage**, **Users and groups**, click **Add user/group**. All users that have given permission to the app will appear here with Default Access granted. Add additional permissions (`app_reader`, `app_writer`, etc.) for whatever users you want to have access, `test.account@mycompany.com` in this example.

Create an application registration for the Stardog client (see: [here](access-token-passthrough-mode.md#how-to-register-the-launchpad-application). Provide a **Name** for the application (e.g. `Example Launchpad Stardog Client`). Select the supported account types, for example, **Accounts in this organizational directory only**. Create a **Web redirect URI** of `<BASE_URL>/oauth/azure/redirect` (e.g. `http://localhost:8080/oauth/azure/redirect`). Note this value of **<BASE_URL>** must match the value set in the Docker Compose `.env` file, which is the URL you will use to access Launchpad.

After clicking the **Register** button:  
Make note of the Application (client) ID (`05f84535-xxxx-1111-879b-90df20f7168a`).
Client Secret: Under Certificates & secrets, create a New client secret (`83f8Q~wxyz6789e4LlvQOZSYnoHbx50QlpqvGdCo`).

### Update Stardog Application to Accept Client Connections
Go to the App Registration for Snowflake Client Resource (e.g. `Example Stardog Resource`).
Under **Expose an API**, use the default suggested **Application ID URI** (`api://<client-id>` where `<client-id>` is the **Application (client) ID** noted in Step 1 (e.g. `api://ad6ad6ae-1234-46c2-8137-caac1396e658`) and add a `user_login` (this name isn't special, it just needs to be used consistently) scope. Set the **Who can consent?** option to **Admins and users**, enter the required display names and descriptions, and click the **Add scope** button. This creates a scope named, for example, `api://ad6ad6ae-1234-46c2-8137-caac1396e658/user_login`.

Token Version 2: Under **Manifest**, find the `api` object. Change the value for `requestedAccessTokenVersion` from `null` to `2` (this used to be a top-level `accessTokenAcceptedVersion` field). This is required for the access token's `iss` (issuer) claim to match the setting we'll use in the Stardog server's `jwt.conf` file.

Substituting the path to your Stardog properties file for `<STARDOG_HOME>`, add these lines to your `<STARDOG_HOME>/stardog.properties` file:
```
jwt.disable=false
jwt.conf=<STARDOG_HOME>/jwt.yaml
```

Create a `<STARDOG_HOME>/jwt.yaml` file with these contents:
```
issuers:
  <AZURE_AD_AUTHORITY_URL>:
    usernameField: preferred_username
    audience: <OAUTH_CLIENT_ID>
    clientSecret: <OAUTH_CLIENT_SECRET>
    autoCreateUsers: true
    roleMappingSource: token
    rolesField: roles
    allowedGroupIdentifiers: [app_reader,app_writer]
    algorithms:
      RS256:
        keyUrl: <AZURE_AD_JWS_KEY_ENDPOINT>
```

In our example:
```
issuers:
  https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/v2.0:
    usernameField: preferred_username
    audience: ad6ad6ae-1234-46c2-8137-caac1396e658
    clientSecret: GcN8Q~8u4wAjMkduDog4T2eh4E6vQvwASRRXPdy~
    autoCreateUsers: true
    roleMappingSource: token
    rolesField: roles
    allowedGroupIdentifiers: [app_reader,app_writer]
    algorithms:
      RS256:
        keyUrl: https://login.microsoftonline.com/ff24ca66-abcd-1234-8acf-43f2635ada42/discovery/v2.0/keys
```

Restart the Stardog server.

### Launchpad Environment Settings
Copy the contents of <https://github.com/stardog-union/launchpad-docs/tree/main/azure> to a folder from which you will launch the Launchpad docker container. Update the (hidden) `.env` file as follows:  
For the line `AZURE_CLIENT_ID=<client-id>`, replace `<client-id>` with the value noted in the instructions for registering the Launchpad Application.
```
AZURE_CLIENT_ID=05f84535-xxxx-1111-879b-90df20f7168a
```
For the line `AZURE_CLIENT_SECRET=<client-secret>`, replace `<client-secret>` with the value noted in the instructions for registering the Launchpad Application.
```
AZURE_CLIENT_SECRET=83f8Q~wxyz6789e4LlvQOZSYnoHbx50QlpqvGdCo
```
Un-comment the line `AZURE_AUTH_TOKEN_TYPE` and set its value to `access_token`.
```
AZURE_AUTH_TOKEN_TYPE=access_token
```
Un-comment the line `AZURE_STARDOG_SCOPE` and replace `<Stardog-Client-ID>` with the value noted in the instructions for registering the Stardog Application, so that it is set to `AZURE_STARDOG_SCOPE=api://<Stardog-Client-ID>/user_login`.
```
AZURE_STARDOG_SCOPE=api://4c2aad88-0d08-4623-959b-62961936dc16/user_login
```

Install and launch Launchpad:
```
$ docker login stardog-stardog-apps.jfrog.io

$ docker pull stardog-stardog-apps.jfrog.io/launchpad:current

$ docker-compose up
```

The last line of the nearly 200 console message lines should be:
```
launchpad-1  | running "unix_signal:15 gracefully_kill_them_all" (master-start)...
```

Logout from your Microsoft account: [https://login.microsoftonline.com/organizations/oauth2/v2.0/logout]

Navigate the the Launchpad URL: [http://localhost:8080/]

You should be asked to choose between username/password and Microsoft (if not, delete cookies from localhost from your browser and try again).

Login as the test user (`test.account@mycompany.com`). You should be asked to grant permission to Launchpad to access Stardog resources on your behalf:

> test.account@mycompany.com  
> Permissions requested (1 of 2 apps)  
> Example Launchpad Stardog Client  
> App info  
> This application is not published by Microsoft.  
> This app would like to:  
> user_login (Example Stardog Resource)  
> View your basic profile  
> Maintain access to data you have given it access to

and:
> test.account@mycompany.com  
> Permissions requested (2 of 2 apps)  
> Example Stardog Resource  
> mycompany.onmicrosoft.com  
> This application is not published by Microsoft.  
> This app would like to:  
> View users' basic profile  
> Maintain access to data you have given it access to

You should see an app launcher panel (rather than a dianostics page).

Go to Studio.

Create a datasource with these properties:
```
jdbc.url=jdbc:snowflake://12345.us-east-1.snowflakecomputing.com/?db=VGDB&warehouse=COMPUTE_WH
jdbc.username=VGTEST
jdbc.password=*************
jdbc.driver=net.snowflake.client.jdbc.SnowflakeDriver
jdbc.passthrough=OAUTH
jdbc.passthrough.scope=api://08bcca47-1234-48ae-b92e-24f52263d55f/session:scope:sysadmin
passthrough.authenticator=oauth
passthrough.token={ACCESS_TOKEN}
```

Create a virtual graph named `snowflake_passthrough` with the same mappings as our `snowflake_oauth` virtual graph:
```
MAPPING
FROM SQL {
  SELECT CURRENT_USER the_user, *
  FROM PUBLIC."Student_Sport"
}
TO {
  ?s <urn:sport> ?Sport .
  ?s <urn:asSeenBy> ?the_user .
}
WHERE {
  BIND(TEMPLATE("urn:person:{Student}") AS ?s) 
}
```

Run a query (note the new virtual graph name in the query):
```
select * from <virtual://snowflake_passthrough> {
  ?s ?p ?o
}
```

Should get the same results, however, the `urn:asSeenBy` results should match the current user:
```
+------------------+--------------+--------------------------------+
|        s         |      p       |               o                |   
+------------------+--------------+--------------------------------+
| urn:person:Venus | urn:asSeenBy | "test.account@mycompany.com" |
| urn:person:Venus | urn:sport    | "Tennis"                       |   
| urn:person:SMITH | urn:asSeenBy | "test.account@mycompany.com" |
| urn:person:SMITH | urn:sport    | "Hackeysack"                   |   
+------------------+--------------+--------------------------------+

Query returned 4 results in 00:00:00.694
```

