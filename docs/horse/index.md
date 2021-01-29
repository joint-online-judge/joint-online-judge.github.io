# Introduction

## Requirements

+ Python >= 3.7 with pip (required), virtualenv (recommended)
+ mongodb >= 3.5
+ rabbitmq
+ redis

They can be easily installed on most operating systems.

=== "Debian/Ubuntu"
    
    Note that  python3.7+ is available as `python3` in Debian 10+ or Ubuntu 20.04+ 
    in the official package repository. 

    If you are using an OS with a lower version, you need to change the commands below.
    
    ```bash
    sudo apt update
    sudo apt install python3 python3-pip python3-virtualenv
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

    I don't have a Mac, maybe someone else will complete it.