Topics to be included in a upcoming release of the [README.md](README.md).

* Zip for composer performance by `sudo apt install zip unzip php-zip`
* add convenience aliases to `~/.bash_aliases`
  * add [sail alias](https://laravel.com/docs/8.x/sail#configuring-a-bash-alias) `alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'`
  * alias for proxy to avoid moving between directories `alias nproxy='docker compose -f ~/nginx-proxy/docker-compose.yml'`
* using the proxy to add a `proxy.test` domain for itself to `host.docker.internal:81` 
* note about the possibility to add volumes for package development
  * `~/code` and `~/code/packages` 
  * `.:/var/www/html` and `../packages:/var/www/packages` to keep relative paths in sync on host and container
* note about setting up the windows gpg path in the WSL `~/.gitconfig` for signing commits
  * `[gpg] program = "/mnt/c/Program Files (x86)/GnuPG/bin/gpg.exe"`
* note about uninstalling (deactivate `VirtualMachinePlatform` and `Microsoft-Windows-Subsystem-Linux`)
