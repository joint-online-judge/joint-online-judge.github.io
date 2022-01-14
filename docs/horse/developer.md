# Developer's Guide

As a developer of `horse`, usually you need to start the `horse` FastAPI server locally 
to test the code you write.

`horse` supports Windows, Linux and macOS, you can setup all services one by one 
manually. However, this is very complicated and hugely discouraged.

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

    **WARNING**: DO NOT run the start.sh mentioned below or start any docker container in Windows, do these in WSL2. Or the data in volumes will get lost during development.

=== "Linux"

    You can install [Docker Engine](https://docs.docker.com/engine/install/) 
    directly. There is a guide for most distributions, for example, the guide for 
    [Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

    For Docker Compose, please read this [guide](https://docs.docker.com/compose/install/) 
    to install it.

## Clone Repositories

You can create a directory `joj` and clone the repositories.

!!! quote "Command"

    ```bash
    mkdir -p joj && cd joj
    git clone git@github.com:joint-online-judge/horse.git
    git clone git@github.com:joint-online-judge/joj-deploy-lite.git
    ```


## Setup All Services Locally

### Related files

#### docker-compose.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose.yml) contains all services of JOJ 2.0. Usually you don't need to modify it. You can overwrite the settings in other files.

#### docker-compose-ui.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose-ui.yml) contains web uis for the services (redis, postgres, etc.).

The usernames and passwords are set in the compose files, you can use them to login.

#### docker-compose-dev.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose-dev.yml) contains some development settings. Usually you don't need to modify it as well because the environment variables can be set in the `.env` file.

???+ example

    This is a our shipped `docker-compose-dev.yml` file:

    ``` yaml title="docker-compose-dev.yml" linenums="1"
    version: '3'
    services:
      horse:
        image: ghcr.io/joint-online-judge/horse:test
        environment:
          DEBUG: "true"
          OAUTH_GITHUB: "true"
          OAUTH_GITHUB_ID: ${OAUTH_GITHUB_ID:-}
          OAUTH_GITHUB_SECRET: ${OAUTH_GITHUB_SECRET:-}
        volumes:
          - ${HORSE_SRC}/joj/horse:/root/joj/horse
          - ${HORSE_SRC}/migrations:/root/migrations
    ```

    We enable the debug mode for auto reloading and set the GitHub OAuth2 client id and secret.

    You are recommended to overwrite the environment variables `OAUTH_GITHUB_ID`, `OAUTH_GITHUB_SECRET` and `HORSE_SRC` in the `.env file. 


#### .env (Optional)

Create a file called `.env` in the folder to override the environments in `docker-compose.yml` and `docker-compose-dev.yml`.

???+ example

    You need to override `OAUTH_GITHUB_ID`, `OAUTH_GITHUB_SECRET` and `HORSE_SRC`, which is set in `docker-compose-dev.yml`.

    ``` dotenv title=".env" linenums="1"
    OAUTH_GITHUB_ID=78****************30
    OAUTH_GITHUB_SECRET=2b************************************95
    HORSE_SRC=/path/to/horse
    ```

!!! note

    For inner developers, please find the client id and secret in the `deploy` channel on `Slack`.

If you don't use the `.env` file, you can also hard-code it in `docker-compose-dev.yml`.

Check the README file in `joj-deploy-lite`, you can set `HORSE_SRC` in the `.env` file for convenience.

### Deployment Command

In the `joj-deploy-lite` repository, you can find a script `start.sh`, which can be used to build and start the services automatically.

!!! quote "Command"

    ```bash
    bash start.sh dev
    ```

## Setup Horse Locally and Connect to the Remote Staging Server

### Deployment Command

!!! quote "Command"

    ```bash
    bash start.sh stage
    ```

## Setup Environment for IntelliSense & Testing

If you do not want to attach to the Docker container each time for an full Python environment that can let your code editor provide intellisense, you may need to install the related Python packages via [Poetry](https://python-poetry.org/).

We use [pre-commit](https://pre-commit.com/) to do pre-commit checks, do not forget to install it before committing.

!!! quote "Command"

    ```bash
    curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python - # install poetry
    poetry install -E test # install all packages
    poetry run pre-commit install # install pre-commit hooks
    ```

We also use [pytest](https://docs.pytest.org/) for testing. You can run the test inside the Docker container, only after you have setup all services locally.

!!! quote "Command"

    ```bash
    docker exec -it `docker ps -q --filter ancestor=ghcr.io/joint-online-judge/horse:test` pytest
    ```
