# Developer's Guide

As a developer of `horse`, usually you need to start the `horse` FastAPI server locally 
to test the code you write.

`horse` supports Windows, Linux and macOS, you can follow the instructions in the 
introduction  to start the server, and then setup all services one by one manually. 
However, this is very complicated and hugely discouraged.

We recommend you to use [Docker](https://docs.docker.com/get-started/overview/) and 
[Docker Compose](https://docs.docker.com/compose/) to deploy the services.


## Install Docker and Docker Compose


=== "Windows/macOS"

    You can use [Docker Desktop](https://docs.docker.com/desktop/).

    On Windows, it's recommended to use Docker Desktop in WSL2 mode, and you can enable
    the WSL2 integration. 

    You don't need to install Docker Compose because it is already shipped with 
    Docker Desktop. However, for WSL users, Docker Compose is installed on Windows,
    not WSL, you may need to install it manually (refer to the guide for Linux).
    

=== "Linux"

    You can install [Docker Engine](https://docs.docker.com/engine/install/) 
    directly. There is a guide for most distributions, for example, the guide for 
    [Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

    For Docker Compose, please read this [guide](https://docs.docker.com/compose/install/) 
    to install it.



## Setup All Services Locally

### Deployment (All Services)

Create a folder which will contain the compose files and an environment file.

#### docker-compose.yml

Create a file called `docker-compose.yml` in the folder.

??? summary "Copy and Paste"
    
    ``` yaml title="docker-compose.yml" linenums="1"
    --8<-- "docs/horse/docker-compose.yml"
    ```


#### docker-compose-dev.yml (Optional, Recommended)

Create a file called `docker-compose-dev.yml` in the folder. You can override some settings in this file.

???+ example

    This is a recommended `docker-compose-dev.yml` file:

    ``` yaml title="docker-compose-dev.yml" linenums="1"
    version: '3'
    services:
      horse:
        image: ghcr.io/joint-online-judge/horse:latest
        environment:
          DEBUG: "true"
          OAUTH_GITHUB: "true"
          OAUTH_GITHUB_ID: 78****************30
          OAUTH_GITHUB_SECRET: 2b************************************95
        volumes:
          - ${HORSE_SRC}/joj/horse:/root/joj/horse
          - ${HORSE_SRC}/migrations:/root/migrations
    ```

We enable the debug mode for auto reloading and set the GitHub OAuth2 client id and secret.

!!! note

    For inner developers, please find the client id and secret in the `deploy` channel on `Slack`.

#### .env (Optional)

Create a file called `.env` in the folder to override the environments in `docker-compose.yml` and `docker-compose-dev.yml`.

???+ example

    You need to override `HORSE_SRC`, which is set in `docker-compose-dev.yml`.

    ``` dotenv title=".env" linenums="1"
    HORSE_SRC=/the/path/to/your/source/directory/of/horse
    ```

If you don't use the `.env` file, you can also hard-code it in `docker-compose-dev.yml`.

#### Deployment Command

!!! quote "Command"

    ```bash
    docker-compose -f docker-compose.yml -f docker-compose-dev.yml up -d
    ```


### Deployment (All Services + Web UI)

We also configured some web ui for each service (redis, postgres, etc.)
You can add the file `docker-compose-ui.yml` in the folder to enable them.

??? summary "Copy and Paste"

    ``` yaml title="docker-compose-ui.yml" linenums="1"
    --8<-- "docs/horse/docker-compose-ui.yml"
    ```

The usernames and passwords are set in the compose files, you can use them to login.

Change the deployment command to

!!! quote "Command"

    ```bash
    docker-compose -f docker-compose.yml -f docker-compose-dev.yml -f docker-compose-ui.yml up -d
    ```


## Connect to the Services on the Staging Server

bmm will complete this part?
