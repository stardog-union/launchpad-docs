# Azure AD Client Credentials: Using a Certificate

When you configure Launchpad to use Azure AD as an identity provider, you create a registered application with the [Microsoft Identity Platform](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#register-an-application) and you configure Launchpad to connect to the registered app when authenticating a user.

One of the requirements for using a registered application in this manner is that the requesting application (Launchpad in this case) must present a credential as a means of authenticating itself to Azure. The credential can be a client secret (string), a certificate, or federated identity credentials. (See this [doc](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-credentials) for more details.)

The two examples ([basic mode](./README.md) and [access token passthrough mode](./access-token-passthrough-mode.md)) for configuring Launchpad with Azure AD both describe configuring Launchpad to present a client secret credential. This document describes how to use a client certificate credential instead.

## A Little More Detail

The basic flow is as follows:

1. Generate a certificate and its corresponding private key (also called a *key pair*). Note that the certificate is sometimes referred to as the *public key*.
1. Upload the certificate to Azure ([instructions](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app#add-a-certificate)).
1. Configure Launchpad to use the private key along with the thumbprint of the certificate.

When Launchpad sends an OIDC request to Azure to authenticate a user, Launchpad sends the request as a JSON Web Token (JWT), including an additional assertion in the JWT, which is that the JWT is signed by the private key. The JWT header also includes the certificate's thumbprint so that Azure can select the certificate being used to verify the request.

Microsoft provides a good description of the certificate credential mechanism (called `private_key_jwt`) in their [docs](https://learn.microsoft.com/en-us/azure/active-directory/develop/certificate-credentials).

## Configuration Steps

### Generate a Certificate

The following command generates a certificate and private key that can be used for testing. For production, you should use a key pair generated from a trusted certificate authority.

```bash
% openssl req -nodes -x509 -days 30 -newkey rsa:2048 -keyout test-launchpad.key.pem -out test-launchpad.cert.pem
```

Follow the prompts, entering the requested information to create a Distinguished Name for your certificate. In this example, the key pair will expire after 30 days.

   > **Note:**
   > The private key must not be password protected. If prompted for a password when generating the key pair to use with Launchpad, leave the password blank.

You will also need the thumbprint of the certificate when you configure Launchpad. You can provide this either by giving Launchpad access to the certificate file or by computing the thumbprint and providing it in the Launchpad `.env` file.

The following command computes the thumbprint of the certificate generated above.

```bash
% openssl x509 -noout -fingerprint -sha1 -inform pem -in test-launchpad.cert.pem
```

### Upload the Certificate to Azure

1. Under **Certificates & secrets**, click the **Certificates** tab.
1. Click the **Upload certificate** button, select the `test-launchpad.cert.pem` file generated in the previous steps, and then click the **Add** button.

Make note of the thumbprint displayed in the Azure web UI for the newly-added certificate. The **Certificates & secrets** page does not display the full thumbprint string, but it shows enough for you to compare to the thumprint generated in the previous steps by `openssl`, ensuring that the certificate is configured properly.

### Configure Launchpad

We provide an example Docker [configuration .env file](./.env) that you will need to update as follows:

   - Remove (or comment out) the line `AZURE_CLIENT_SECRET=<client-secret>`. If you choose to comment out the line, simply insert a `#` character at the beginning of the line.
   - Un-comment `AZURE_CLIENT_PRIVATE_KEY_FILE=/launchpad.key.pem`, replacing `launchpad.key.pem` with the path to the private key file that corresponds to the certificate that you uploaded in the previous section. For the test certificate generated above, use `/test-launchpad.key.pem`.

In addition to setting the path to the private key file, you must also provide the thumbprint of the certificate. You do this by un-commenting either the `AZURE_CLIENT_CERTIFICATE_THUMBPRINT` or `AZURE_CLIENT_CERTIFICATE_FILE` line in the `.env` file and setting its value appropriately. If both are present, the `AZURE_CLIENT_CERTIFICATE_THUMBPRINT` value is used (and the `AZURE_CLIENT_CERTIFICATE_FILE` value is ignored).

If you have the certificate's thumbprint (say, by using the `openssl` command in the previous section), then use it as the value of the `AZURE_CLIENT_CERTIFICATE_THUMBPRINT` variable. If you do not have an easy way to generate the thumbprint, then provide the path to the certificate file; for the test certificate generated in the previous section, you would set `AZURE_CLIENT_CERTIFICATE_FILE=/test-launchpad.cert.pem`.

When supplying the path to the private key file (and optionally the certificate file), the path must be one that is available in the Docker container where Launchpad is running. To accomplish this, edit the [docker-compose config file](./docker-compose.yml) by un-commenting the appropriate line(s) in the `volumes:` section of `docker-compose.yml`.

#### Example: Specify the Thumbprint

Using the test key pair from the previous section, if the thumbprint string printed by the `openssl x509 -noout -fingerprint -sha1 -inform pem -in test-launchpad.cert.pem` command is `SHA1 Fingerprint=7D:34:92:AC:E9:16:62:41:49:39:A0:3A:06:58:2A:AF:D2:54:5A:6B`, then the `.env` would contain the following:

   ```properties
   AZURE_AUTH_ENABLED=true
   AZURE_CLIENT_ID=aa11bb22-cc33-dd44-ee55-6666ffff7777
   #AZURE_CLIENT_SECRET=
   AZURE_TENANT=organizations
   AZURE_CLIENT_PRIVATE_KEY_FILE=/test-launchpad.key.pem
   AZURE_CLIENT_CERTIFICATE_THUMBPRINT="SHA1 Fingerprint=7D:34:92:AC:E9:16:62:41:49:39:A0:3A:06:58:2A:AF:D2:54:5A:6B"
   ```

and the `docker-compose.yml` file would contain:

   ```yml
   volumes:
      - ./jwk:/jwk
      - ./test-launchpad.key.pem:/test-launchpad.key.pem
   ```

#### Example: Specify the Certificate File

Using the test key pair from the previous section, the `.env` would contain the following:

   ```properties
   AZURE_AUTH_ENABLED=true
   AZURE_CLIENT_ID=aa11bb22-cc33-dd44-ee55-6666ffff7777
   #AZURE_CLIENT_SECRET=
   AZURE_TENANT=organizations
   AZURE_CLIENT_PRIVATE_KEY_FILE=/test-launchpad.key.pem
   AZURE_CLIENT_CERTIFICATE_FILE=/test-launchpad.cert.pem
   ```

and the `docker-compose.yml` file would contain:

   ```yml
   volumes:
      - ./jwk:/jwk
      - ./test-launchpad.key.pem:/test-launchpad.key.pem
      - ./test-launchpad.cert.pem:/test-launchpad.cert.pem
   ```

   > **Note:**
   > If the `AZURE_CLIENT_SECRET` value is set in the `.env` file, then the client secret is used as the credential when Launchpad authenticates iteself to Azure, and the variables related to configuring the client certificate are ignored.

## Run the Example

1. Execute the following command from this directory to bring up the Launchpad service.

   ```bash
   docker-compose up
   ```

2. Visit [http://localhost:8080](http://localhost:8080) in your browser.

3. Click the "Sign in with Microsoft" button.

See [README.md Run the Example](./README.md#run-the-example) for more info.