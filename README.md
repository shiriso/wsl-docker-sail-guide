# Setting up WSL, Docker and Sail for local development

## Introduction
Ever wondered what your preferred development solution could look like?

Maybe you already used the good old [Vagrant](https://www.vagrantup.com) and [Homestead](https://laravel.com/docs/8.x/homestead) just to get disappointed by its performance over time.

So you tried to get into [Docker](https://www.docker.com) as the new cool tool but felt overwhelmed by its possibilities and amount of configuration needed?

Then you found out about [Laradock](https://laradock.io) and [Laragon](https://laragon.org) as the cool kids on the block to take care of the whole dockerfile/docker-compose you never got a hang on.

Guess what? It seems like [@laravelphp](https://twitter.com/laravel) felt the same, so they released [Laravel Sail](https://github.com/laravel/sail) to take care of the dockerfiles
for you as well, with everything needed for Laravel. And who would know better what is needed to run Laravel?

While this guide mostly focuses on Windows environments, the general explanations may apply to Linux / macOS and Unix based hosts as well. Just consider to Skip the WSL-related parts.

Also, even though Windows 10 supports WSL2, the implementation feels vastly more integrated when running Windows 11. If you get the chance, try it out to see the difference!
Using Windows Terminal as default shell, being able to access the container's storage simply through `\\wsl$\`, or by just clicking it in the explorer feels so much better.

## Table of contents
- [Introduction](#introduction)
- [Installing WSL & Docker](#installing-wsl--docker)
    - [Requirements](#Requirements)
    - [WSL Container Setup](#wsl-container-setup)
    - [Upgrading Ubuntu](#upgrade-ubuntu)
        - [Default User](#default-wsl-user)
- [Installing Dependencies](#installing-dependencies)
    - [PHP](#php)
        - [Default CLI Version](#default-php-for-cli)
    - [Composer](#composer)
- [Setup Applications](#set-up-the-applications)
    - [Updating the Environment](#environment)
        - [Modifying the Ports](#modifying-the-ports)
        - [Modifying the Service Container](#modifying-the-service-container)
- [Domains](#managing-domains)
    - [Choosing the TLD](#choosing-the-correct-tld)
        - [.test](#test)
        - [.localhost](#localhost)
        - [.local](#why-not-local)
        - [Why not .dev?](#why-not-dev)
    - [Setting up a Local Domain](#setting-up-a-local-domain)
    - [Multiple Domains](#multiple-domains)
        - [Nginx Proxy Manager](#nginx-proxy-manager)
            - [Setup](#setup)
            - [Start](#start-the-proxy)
            - [Configure](#configure)
- [Optional Convenience Features](#optional-convenience-features)
    - [SSH Keys](#ssh-keys)
        - [Sync Keys](#sync-keys)
        - [Key Manager](#key-manager)
    - [Start Directory](#start-in-home)
- [Cheatsheet](#cheatsheet)
    - [WSL](#wsl)

## Installing WSL & Docker

### Requirements
To enable the necessary `Windows Hypervisor` and the `Windows Subsystem for Linux` start a PowerShell with administrator privileges.

Enable the VirtualMachinePlatform (Windows Hypervisor) required for WSL:
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Enable the WSL Windows feature:
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

It is highly recommended rebooting your PC at this point, as you may not be able to proceed further with the setup.

#### Install the Following Software
- [WSL Kernel Update Package](https://docs.microsoft.com/de-de/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- Make sure to enable the [Docker backend for WSL 2](https://docs.docker.com/desktop/windows/wsl/#install)

### WSL Container Setup
Open a PowerShell to install and configure your WSL environment by executing the following commands.

Set the WSL default to version 2, to automatically use this version for new distributions:
```
wsl --set-default-version 2
```

Install your desired Linux distribution and follow the installer:
```
wsl --install -distribution Ubuntu
```

Check your installed WSL environments:
```
wsl --list --verbose
```

Configure the default distribution:
```
wsl --set-default <name from previous command> <desired wsl version>
```

For example, when installing the latest Ubuntu while using WSL2, we would use this command:
```
wsl --set-default Ubuntu 2
```

To start your freshly installed Ubuntu, just call:
```
wsl
```

This is where the magic happens. This will boot your container, connect you to it and start you the default prompt of your all shiny new Ubuntu.

## Upgrade Ubuntu
As only Ubuntu's LTS releases are available out of the box, you may want to upgrade if you want more up-to-date packages,
but as we manually install everything we need, it should not be necessary.

But as many developers tend to like cutting edge technologies, here is how you can do it anyway.

Open the `/etc/update-manager/release-upgrades` file and change the last line from `prompt=lts` to `prompt=normal`:
```
sudo nano /etc/update-manager/release-upgrades
```

Remove `snapd` to avoid issues, as the WSL Ubuntu is not booted by `systemd` and does not support it.
```
sudo apt remove snapd
```
> Improvised snapshots can be accomplished by exporting and importing the WSL container itself.

The following steps may have to be repeated, since upgrading multiple versions of linux at once is not supported.
For example, if you want to upgrade to 21.04 and 21.10 afterwards, you will need to run the following commands twice.

Update and upgrade your local dependencies:
```
sudo apt update && sudo apt upgrade
```

Run the actual upgrade:
```
sudo do-release-upgrade
```

Stop your WSL distribution through your PowerShell terminal:
```
wsl --terminate Ubuntu
```

Start the container again (`--distribution Ubuntu` may be omitted if that's your default container):
```
wsl --distribution Ubuntu
```

### Default WSL User

If you upgraded your Ubuntu, you might notice that you will be logged in as root when connecting to the container.
To get Ubuntu back to its default behaviour, you need to edit or create the wsl.conf:
```
nano /etc/wsl.conf
```

Add the following code to the file and replace `username` with your existing linux user:
```
[user]
default=username
```

## Installing Dependencies
If the `Dockerfiles` from sail are not published into your repository but only exist in the `vendor` directory,
your local WSL (or other *nix) system may need to fulfill all the requirements of your app, or you will not be able to `sail up` prior to `composer install`.

This can become quite troublesome when various projects may have different requirements for php as you would need to switch the default php version before running composer commands.
Because of this we highly recommend publishing these files by using `sail artisan sail:publish`.

Afterwards you will be able to run sail by using the regular `docker compose` commands and use the container itself to work with composer.

If you want to add sail to an existing application and can not prepare the switch by requiring and publishing the files from a different development environment,
you may use the following steps to fulfill the requirements for most applications.

### PHP
Add the external PPA for up-to-date PHP versions:
```
sudo add-apt-repository ppa:ondrej/php
```

First, update the available packages:
```
sudo apt update
```

Install your desired php version with the according extensions.
With most versions you may simply replace the 8.0 with the version you want:
```
sudo apt install -y php8.0-cli php8.0-dev php8.0-pgsql php8.0-sqlite3 php8.0-gd php8.0-curl php8.0-memcached php8.0-imap php8.0-mysql php8.0-mbstring php8.0-xml php8.0-zip php8.0-bcmath php8.0-soap php8.0-intl php8.0-readline php8.0-pcov php8.0-msgpack php8.0-igbinary php8.0-ldap php8.0-redis php8.0-swoole php8.0-xdebug
```

When you install a new version of php, it will automatically register a link from `/usr/bin/php`, and possibly through another to `/etc/alternatives/php`, to the installed version, like for example `/usr/bin/php8.0`.

This means that you can now use the `php` command to use your latest installed version.

To run a command or similar with another version of PHP, you can specify the version by calling `php8.0` or `php8.1`, instead of `php`.

#### Default PHP for CLI
Since tools like composer will use your default cli php version, you may need to change it accordingly to your requirements.

You can switch between versions by adjusting the according symlink...
```
sudo ln -s -f /usr/bin/php8.0 /etc/alternatives/php
```

... or by using `update-alternatives`:
```
sudo update-alternatives --set php /usr/bin/php8.0
```

### Composer
Switch into your home directory:
```
cd ~
```

Download the current composer install script:
```
curl -sS https://getcomposer.org/installer -o composer-setup.php
```

Validate that the download was successful:
```
HASH=`curl -sS https://composer.github.io/installer.sig`
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

Run the installer:
```
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Make sure you installed the current version of composer:
```
composer self-update --2
```

Remove the setup script:
```
rm composer-setup.php
```

Add zip packages to improve performance:
```
sudo apt install zip unzip php-zip
```
> Note: php-zip may potentially be omitted, if you installed a custom php version before as described in [PHP](#php).

## Set up the Applications
While the Laravel documentation may cover some basics at [Getting started on Windows](https://laravel.com/docs/8.x/installation#getting-started-on-windows)
and [Installing Sail into existing applications](https://laravel.com/docs/8.x/sail#installing-sail-into-existing-applications),
it is lacking some advanced information which might be hard to catch up with.

Did you ever need to run multiple projects at the same time? Maybe when you developed a microservice and its integration in another application at the same time?

Although Sail itself is already prepared to run multiple instances simultaneously, this is not its default behavior.

That is where we have got you covered as well.

As each port can only be exposed once by docker, it is not possible to run various containers at port `80` for http,
which would be the default for Sail, which makes perfectly sense, as you can simply open `localhost` to take a look at your awesome application.
Docker would fail to start a second application as the port is already in use.

How do we fix that though?

### Environment
If you ever took a look at the `docker-compose.yml`, which will be provided by sail, you may have noticed these lines defining the ports to expose.

At the `laravel.test` service container:
```
- '${APP_PORT:-80}:80'
```

At the databases `mysql` service container:
```
- '${FORWARD_DB_PORT:-3306}:3306'
```

The syntax for Docker follows the following pattern: `'[Expose Port]:[Internal Port]'`,
where the `Expose Port` will make the container available at your `localhost`,
while the right side is the internal port used within the container's internal network.

On the left side we've got an expression like `${APP_PORT:-80}`, which means that it will use the given `APP_PORT` variable or use port `80` as default if none is set.

As the right side is only used in the container's internal network, it's entirely fine to use the same ports in every `docker-compose.yml`.

If you want to know more about variables in compose, check the [Docker Documentation for Environment variables in Compose](https://docs.docker.com/compose/environment-variables).

#### Modifying the Ports
Since we now know that Laravel Sail supports modifying the exposed ports, the only questions left are: How does Docker retrieve these variables and how can you set them?

While it is possible to define environment variables when calling `docker compose`, defining them in the `docker-compose.yml` itself or pass a `.env` file to use,
the easiest solution is the one which gets used automatically:

**It will simply use your applications already existing `.env` file!**

So just open your existing `.env` file and add the following keys:
```
APP_PORT=41000
FORWARD_DB_PORT=41001
```

The used ports are just examples. Try to avoid conflicts with reserved and registered ports since you might experience other applications behaving in an unexpected manner
or get issues when trying to boot your Docker container. In doubt check the [List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).

#### Modifying the Service Container

Should you experience an issue running actions on the Laravel service container using Sail, it is most likely due to your application's service container name not being `laravel.test`.
You can easily find the name of your service container when taking a look in the `docker-compose.yml`.

If you are using a different name, set the according key in your `.env` file like this:
```
APP_SERVICE=laravel.test
```

## Managing Domains
Docker exposes the running containers based on their given port on `localhost`, as defined in `docker-compose.yml`.
In addition, we might want to access our projects using domains, which are way more convenient and easier to remember.

### Choosing the Correct TLD
When setting up your local domain, you should keep in mind the intended use of top level domains as defined in their according [RFCs](https://datatracker.ietf.org/doc/html/rfc2606).

#### .test
`.test` should usually be your go-to TLD for local development, as it is protected by an RFC and will never be available to register domains on the internet.
It can be routed locally inside networks though, when sharing an application for preview purposes or similar.

#### .localhost
As `.localhost` is always forced to represent a local loop to your localhost, it could be used for local development, but could never be shared within your network.
This could be used as go-to for local development, never has to be shared with others in any way.

#### Why not .local?
The `.local` TLD is reserved for internal networks, and will thus also never be available for internet domain registration, and as such may be used for local development.
But since tools like zeroconf and bonjour use it automatically and may already have a service assigned to `[computer name].local`
it is recommended to not use that domain for local development.

#### Why not .dev?
Although developers have often used `.dev` in the past, it is not recommended doing so, as `.dev` is an actually existing TLD owned by Google,
and it is also enforced to run HTTPS for example, which means there is a high probability browsers might block your local requests, when there is no valid certificate.
In addition, you might encounter conflicts trying to visit actual, registered domains and their according websites or services.

### Setting up a Local Domain
Open a console or terminal on your Windows with administrator privileges and edit the hosts-file.
```
notepad C:\Windows\System32\drivers\etc\hosts
```

Add a line pointing your custom domain to `localhost` (`127.0.0.1`).
```
127.0.0.1	domain.test
```

The site can then be visited by using `domain.test` in the browser if the container uses the default port.

If a different port has been exposed for the container, you need to attach it to the domain like `domain.test:54321`.
However, as you would need to remember the port, this would defeat the point of using domains in the first place.

### Multiple Domains
When trying to run multiple containers at once, you need to use ports as only one container can listen to the same port at the same time.

Since the name resolution through the `hosts` file does not cover ports, we will need to use a more sophisticated solution.

For example, we could push the boundaries a bit by using another container with a proxy to deliver each container through a unique domain.

#### Nginx Proxy Manager
[Nginx Proxy Manager](https://github.com/jc21/nginx-proxy-manager) is a software which takes care of configuring an instance of nginx to your redirection needs.
While this is already convenient, there is even an existing image for docker, which we can use as proxy for our containers.

##### Setup
Go to your home directory and create a new directory for the proxy:
```
cd ~
mkdir nginx-proxy
cd nginx-proxy
```

Create a `docker-compose.yml` inside the directory:
```
nano docker-compose.yml
```

Insert the following contents into the file:
```
version: "3"
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: always
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
      DISABLE_IPV6: 'true'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

As we use SQLite, the entire configuration will persist in the shared data volume, even when we reboot the container.

##### Start the Proxy
To start the Proxy, simply start the container in detached mode.
Running it in detached mode allows you to continue using the shell after starting the container:
```
docker compose up -d
```

If you want to automatically start the proxy once you connect to the wsl container you can adjust your `~/.bashrc` accordingly by adding the following line.
```
docker compose -f ~/nginx-proxy/docker-compose.yml up -d
```

##### Configure
Open `http://localhost:81` in your browser to open the administration.
Set the credentials you want to use.

Default Credentials:
```
Mail: admin@example.com
Password: changeme
```

Select `Hosts` in the top navigation, open the `Proxy Hosts` page and add a new one, by selecting `Add Proxy Host` on the top right.

Set your desired domain and fill in the following fields, where the `APP_PORT` should match the value of the same key in your projects `.env` file:
```
Forward Hostname / IP: host.docker.internal
Forward Port: 41000 / <APP_PORT>
```
Port `41000` is only used here to illustrate our example in [Modifying the Ports](#modifying-the-ports).

Keep in mind that every domain that you want to use within the `Nginx Proxy Manager` still needs to be added to your `hosts` file, each simply pointing to `127.0.0.1`.
You should now be able to call `http://domain.test` from within your browser.

> Note: You can also define a `Proxy Host` for the `Nginx Proxy Manager` itself, simply by setting the `Forward Port` to `81`.

## Optional Convenience Features
To improve your experience using WSL and Docker you can do the following adjustments as desired.

### SSH Keys
SSH keys are necessary to perform various actions like git, which you may want to use in your WSL.

#### Sync Keys
To sync the existing SSH keys from your Windows machine, edit the `.bashrc` in your home directory:
```
nano ~/.bashrc
```

Add the following lines at the end of the file:
```
# Import your existing ssh-keys from Windows when logging in into the container.
# As the WSL bash should start in your mounted user directory /mnt/c/Users/[user], 
# we can just copy it from the current directory into the linux home. 
cp -r ./.ssh ~/.ssh

# Adjust the permissions to assure that the keys are only accessible by the user itself.
chmod 700 $(find ~/.ssh -type d)
chmod 600 $(find ~/.ssh -type f)
```
> When using a different Terminal (e.g. PhpStorm) you may need to use the full path.

#### Key Manager
This is only necessary if you want to perform actions which require an SSH key and do not want to enter your passphrase all the time.

Update the packages and install keychain:
```
sudo apt update
sudo apt install keychain
```

Start keychain for the User when logging in:
```
eval ``keychain --eval --agents ssh <your private key filename>
```

As an example, if you use default names and ED25519 as algorithm:
```
eval ``keychain --eval --agents ssh id_ed25519
```

This can also be added to the `.bashrc`.
If you added it into the `.bashrc`, the passphrase will be requested once the login is performed
and the key will be available until the timeout has exceeded or the container was terminated.

### Start in Home
To automatically start in your home directory when connecting to the container edit the `.bashrc`:
```
nano ~/.bashrc
```

Add the following line at the end of the file:
```
cd ~
```

Keep in mind that this should be placed at the very end of the file, since previous commands may rely on the default start directory.

### Shell Aliases
If you want to use aliases to avoid messing around with paths, options or similar, you can set them automatically for your user.

Create or modify `.bash_aliases` within your home directory:
```
nano ~/.bash_aliases
```

> Aliases will only be available once you created a new session.

#### Laravel Sail Alias
Laravel already prepared a general documentation about Sail and [Configuring a Bash Alias](https://laravel.com/docs/8.x/sail#configuring-a-bash-alias).

Simply add the following alias into your `.bash_aliases`:
```
alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'
```

> Keep in mind that you can only use this alias from within your application's directory and
> a published sail file takes precedence over the one located in the vendor directory.

#### Nginx Proxy Manager Alias
As aliases can also be used to append options, you can also create an alias to manage your proxy without navigating there.

Add the following alias to your `.bash_aliases`:
```
alias nginxproxy='docker compose -f ~/nginx-proxy/docker-compose.yml'
```

You can now use something like `nginxproxy up -d` to start your proxy in detached mode, independent of your current working directory.

## Cheatsheet

### WSL
List all available distributions:
```
wsl --list --online
```

List container status:
```
wsl --list --verbose
```
> The `*` in front will mark your current default distribution:

Stop all WSL containers:
```
wsl --shutdown
```

Stop a specific distribution's container:
```
wsl --terminate <distribution>
```

Start a distribution's container:
```
wsl --distribution <distribution>
```

When starting a distribution, the `--distribution` option may be omitted when starting the default.

To start your default distribution:
```
wsl
```

Configure the WSL default to WSL2:
```
wsl --set-default-version 2
```
> Note: This will only apply to containers added in the future:

Define the default distribution (WSL version 2 is highly recommended!):
```
wsl --set-default <name from previous command> <desired wsl version>
```

Remove a registered distribution:
```
wsl --unregister <distribution>
```

Export the current state of your WSL distribution:
```
wsl --export <distribution> <filename or path>.tar
```

Import the current state of your WSL distribution:
```
wsl --import <distribution> <path for virtual disk> <filename or path>.tar
```
> Note: If you want to run WSL from a different drive than your system default, just run this command
> from the specific location using relative paths, or absolute paths including the drive letter.
>
> * Stop the distribution
> * Export the current state
> * Remove the distribution
> * Import the created .tar-file
>
> As applications like docker only rely on the container's name, you can move it wherever you want.
