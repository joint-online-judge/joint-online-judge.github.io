# Developer's Guide

As a developer of joint-online-judge frontend, check [Cattle's README](https://github.com/joint-online-judge/cattle).

As a developer of joint-online-judge backend & judger, usually you need to start the service locally
to test the code you write.

`horse`, `elephant`, and `tiger` supports Windows, Linux and macOS, you can setup all services one by one
manually. However, this is very complicated and hugely discouraged.

We recommend you to use [Docker](https://docs.docker.com/get-started/overview/) and
[Docker Compose](https://docs.docker.com/compose/) to deploy the services.

## Install Docker and Docker Compose

=== "Windows/macOS"

    You can use [Docker Desktop](https://docs.docker.com/desktop/).

    On Windows, it's recommended to use Docker Desktop in WSL2 mode, and you can **enable
    the WSL2 integration**. Follow the steps [here](https://docs.docker.com/desktop/windows/wsl/),
    while pay special attention to [Enabling Docker support in WSL 2 distros](https://docs.docker.com/desktop/windows/wsl/#enabling-docker-support-in-wsl-2-distros).

    You don't need to install Docker Compose because it is already shipped with
    Docker Desktop.

    !!! warning
        **DO NOT** run the start.sh mentioned below or start any docker container in Windows,
        do these in WSL2. Or the data in volumes will get lost during development.

=== "Linux"

    You can install [Docker Engine](https://docs.docker.com/engine/install/)
    directly. There is a guide for most distributions, for example, the guide for
    [Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

    For Docker Compose, please read this [guide](https://docs.docker.com/compose/install/)
    to install it.

## Clone Repositories

You can create a directory `joint-online-judge` and clone the repositories.

!!! quote "Command"

    ```bash
    mkdir -p joint-online-judge && cd joint-online-judge
    git clone git@github.com:joint-online-judge/horse.git
    git clone git@github.com:joint-online-judge/elephant.git
    git clone git@github.com:joint-online-judge/tiger.git
    git clone git@github.com:joint-online-judge/joj-deploy-lite.git
    cd joj-deploy-lite
    ```

## Setup All Services Locally

### .env

Create a file called `.env` in `joj-deploy-lite` to override the environments in `docker-compose.yml` and `docker-compose-dev.yml`.

???+ example

    You need to override `HORSE_SRC`, `TIGER_SRC`, and `ELEPHANT_SRC` to start the project, other variables are optional.

    ``` dotenv title=".env" linenums="1"
    HORSE_SRC=/abspath/to/horse
    ELEPHANT_SRC=/abspath/to/elephant
    TIGER_SRC=/abspath/to/tiger
    ```

!!! note

    For inner developers, please find the client id and secret in the `deploy` channel on `Slack`.

### Other Related files

#### docker-compose.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose.yml) contains all services of JOJ 2.0. Usually you don't need to modify it. You can overwrite the settings in other files.

It uses port 34765 to run horse, you can access it with <http://127.0.0.1:34765>.

#### docker-compose-ui.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose-ui.yml) contains Web UIs for the services (redis, postgres, etc.).

The usernames and passwords are set in the compose files, you can use them to login.

It uses many extra ports to run Web UIs.

- <http://127.0.0.1:34766>: lakeFS (lakeFS object storage for files)
- <http://127.0.0.1:34767>: Adminer (PostgreSQL database)
- <http://127.0.0.1:34768>: Flower (Celery task queue)
- <http://127.0.0.1:34769>: redis Commander (Redis cache, message broker)
- <http://127.0.0.1:34770>: RabbitMQ (RabbitMQ message queue)
- <http://127.0.0.1:34771>: Dozzle (Log viewer for Docker)

#### docker-compose-dev.yml

[This file](https://github.com/joint-online-judge/joj-deploy-lite/blob/master/docker-compose-dev.yml) contains some development settings. Usually you don't need to modify it as well because the environment variables can be set in the `.env` file.

### Deployment Command

In the `joj-deploy-lite` repository, you can find a script `start.sh`, which can be used to build and start the services automatically.
Be patient, all images have a total size of ~5GB.

For China Mainland developers, you may need a proxy to speedup the whole precedure.

!!! quote "Command"

    ```bash
    ./start.sh dev
    ```

Once the command is finished, check <http://127.0.0.1:34765> and other Web UI ports to ensure it works well.

## Setup Environment for IntelliSense & Pre-commit check & Test

### IntelliSense

If you do not want to attach to the Docker container each time for an full Python environment that can let your code editor provide intellisense, you may need to install the related Python packages via [Poetry](https://python-poetry.org/).

!!! quote "Command"

    ```bash
    curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python - # install poetry
    # change directory to horse or elephant or tiger
    poetry install -E test # install all packages
    poetry run pre-commit install # install pre-commit hooks to poetry environment
    ```

### Pre-commit check

We use [pre-commit](https://pre-commit.com/) to do pre-commit checks, do not forget to install it before committing.

Though you have installed pre-commit to poetry environment in previous step, but to use it,
you need to commit in poetry environment. It is a bit complicated, e.g.,

!!! quote "Command"

    ```bash
    poetry run git commit -m "<commit message>"
    # or enter the poetry shell first
    poetry shell
    git commit -m "<commit message>"
    ```

You can choose to install pre-commit in your local environment by `pip install pre-commit`, then change directory to
horse or elephant or tiger and install the hook by `pre-commit install`.

### Test

We also use [pytest](https://docs.pytest.org/) for testing. You can run the test inside the Docker container,
only after you have setup all services locally.

!!! quote "Command"

    ```bash
    docker exec -it horse pytest
    docker exec -it tiger-1 pytest
    ```

## Create users

Now you can create users in horse and tiger to make it works correctly.

0. Make sure in `joj-deploy-lite`, `./start.sh dev` is run and successfully exits.

1. Go to <http://127.0.0.1:34765/api/v1/docs#/auth/v1_register> and register a new user for your self in horse. You only need to provide email, username, and password in Request Body. e.g. `{"username": "uname", "password": "passwd", "email": "admin@joint.online.judge"}`  
And by clicking "Execute", it will create a new user for you, and if it is the first and the only user, the user will be set as a root user.

2. Go to <http://127.0.0.1:34765/api/v1/docs#/auth/v1_login> and login with username and password you just filled in. For the example above, that is `uname` and `passwd`.

3. Go to <http://127.0.0.1:34765/api/v1/docs#/admin/v1_create_judger> to create judgers. And fill in `TIGER_1_USERNAME`, `TIGER_1_PASSWORD`, `TIGER_1_LAKEFS_USERNAME`, `TIGER_1_LAKEFS_PASSWORD`, `TIGER_2_USERNAME`, `TIGER_2_PASSWORD`, `TIGER_2_LAKEFS_USERNAME`, `TIGER_2_LAKEFS_PASSWORD` in `.env` according to the response.

4. In `joj-deploy-lite`, run `./start.sh dev` again. And wait a few seconds until everything is started and connected.

5. Go to <http://127.0.0.1:34765/api/v1/docs#/admin/v1_admin_list_judgers>, and you will find 2 judgers and both of them have `"isAlive": true`. (**NOTE**: the field you need to check should be `isAlive` instead of `isActive`.) And now you have 2 runnable judgers.
