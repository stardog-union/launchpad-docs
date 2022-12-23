# Basic Authentication Example

The purpose of this example configuration is to demonstrate how to deploy and configure On-Prem for a Stardog instance to allow users to sign in with a Stardog username and password.

![Stardog username and password login](./img/basic.gif)

## How This Works

The way this configuration works is a user provides their Stardog username and password to the login dialog. The application validates the user’s provided credentials and attempts to fetch a JWT token from Stardog. This JWT token from Stardog is then used for all subsequent calls against the Stardog server. For example, when a user is authenticated and then decides to open Stardog Studio, the JWT will be used for all authenticated calls against the Stardog server (e.g. listing all databases in the Stardog server, running a query, etc).

## Prerequisites

- Docker installed
- Docker Compose installed
- A Stardog server running locally on port `5820`. 

  > **Note:**
  > If you have a Stardog server running elsewhere (locally or not), this is fine, just modify the `STARDOG_INTERNAL_ENDPOINT` and `STARDOG_EXTERNAL_ENDPOINT` in the [`.env`](.env) file as needed.


## Stardog Server Requirements

- Stardog server must be v7.8 or above
- The following setting should be set in the Stardog’s server’s [`stardog.properties`](https://docs.stardog.com/operating-stardog/server-administration/server-configuration#stardogproperties) you want to authenticate against.

  ```properties
  jwt.disable=false
  ```

  > **Note**:
  > By default this property is set to `false`, so you can likely omit this.

## Run the Example

1. Execute the following command from this directory to bring up the On-Prem service.

```
docker-compose up -d
```

2. Visit [http://localhost:8080]("http://localhost:8080") in your browser.


