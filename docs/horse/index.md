# Introduction

## Requirements

+ Python >= 3.7 with pip (required), virtualenv (recommended)
+ mongodb >= 3.5
+ rabbitmq
+ redis

They can be easily installed on most operating systems.

=== "Debian/Ubuntu"
    
    Note that python3.7+ is available as `python3` in Debian 10+ or Ubuntu 20.04+ 
    in the official package repository. 
    
    If you are using an OS with a lower version, you need to change the commands below.
    
    If you fail to install mongodb on Debian, referring to <https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/>.
    
    ```bash
    sudo apt update
    sudo apt install python3 python3-pip python3-virtualenv python3-setuptools
    sudo apt install mongodb-server rabbitmq-server redis-server
    ```

=== "Arch Linux"

    Enjoy the simplicity! OKay it seems that Ubuntu is simpler here, 
    but at least you won't meet the OS version issue.
    
    ```bash
    sudo pacman -Syyu
    sudo pacman -S python3 python3-pip python3-virtualenv
    sudo pacman -S rabbitmq redis
    sudo pacaur -S mongodb-bin
    ```

=== "Windows"

    We recommend `chocolatey` to install the dependencies. 
    Ensure that you are using an administrative Powershell, you can install it by
    
    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```
    
    Then run these commands to install everything:
    
    ```powershell
    choco install -y python3
    python -m pip install -U pip
    python -m pip install virtualenv
    choco install -y mongodb rabbitmq redis-64
    ```

=== "macOS"

    We recoment `brew` to install the dependencies. You can install `brew` by
    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
    ```
    
    Then run these commands to install everything:
    
    ```bash
    brew install python3
    brew tap mongodb/brew
    brew install mongodb-community
    brew install rabbitmq redis
    ```

    Start them with
    ```
    brew services start mongodb-community
    brew services start redis
    brew services start rabbitmq
    ```
    
## Setup MongoDB in Replica Mode

We use the "transaction" feature in MongoDB, which is not supported on MongoDB standalone mode. This is because oplog is only enabled in replica mode, and the aborting and committing of a transaction need the oplog.

+ In a production environment, it is recommended to create a (at least) 3-member replica set. 
+ In a development environment, a 1-member replica set can be used.

You can either setup a replica set by command line options, or by a configuration file. For simplicity of usage, here we'll introduce how to modify the configuration file.

=== "Linux"
    A default `/etc/mongod.conf` configuration file is included when using a package manager to install MongoDB.

=== "Windows"
    A default `<install directory>/bin/mongod.cfg` configuration file is included during the installation.

=== "macOS"
    A default `/usr/local/etc/mongod.conf` configuration file is included when installing from MongoDB’s official Homebrew tap.

For basic usage of a 1-member replica set, you can set

```text
replication:
   replSetName: "rs0"
```

Here `rs0` is the name of the replica set. There should be a commented `#replication` in the config file, and you can modify it.

Then you need to restart the `mongod` service.

=== "Linux"
    ```bash
    sudo systemctl stop mongod
    sudo systemctl start mongod
    ```

=== "Windows"
    ```powershell
    net stop MongoDB
    net start MongoDB
    ```

=== "macOS"
    ```bash
    brew services stop mongodb-community
    brew services start mongodb-community
    ```

Finally, in the `mongo` shell, run the following command to initialize it as the master node:

```bash
rs.initiate()
```

The response should look like:

```bash
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "127.0.0.1:27017",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1614414300, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1614414300, 1)
}
```

For more information, check [deploy-replica-set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/){target=_blank}.
