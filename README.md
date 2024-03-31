# Docker NGINX-Proxy Setup

These files create a Docker network using nginx-proxy as a reverse proxy, allowing multiple local domain names to use the same standard 80 or 443 ports.

For example, instead of having to configure each local domain with an unused port, such as `yourdomain.example:8080` and `yourdomain2.example:8081`, the reverse proxy takes care of the ports so you can use clean domain names, such as `yourdomain.example` and `yourdomain2.example`.

## Installation

1. In the parent directory where you want to store your Docker reverse proxy files (such as `/Users/Username/Projects/Assets/Docker/`), run: `git clone git@github.com:jacobcassidy/docker-nginx-proxy-setup.git`.
2. Rename the cloned directory from `docker-nginx-proxy-setup` to the project name you want to use, such as _nginx-proxy-base_.
3. Open the Docker Desktop app so the Docker engine is on. (If you don't have the [Docker Desktop](https://www.docker.com/products/docker-desktop/) app installed on your local machine, install it first.)
4. In the _nginx-proxy-base_ directory you cloned, run: `docker compose up -d` to create and start the Docker container.

## Using HTTPS with local domains

You must add TLS certifications for each domain you want to use. This can be done quickly with the [mkcert](https://github.com/FiloSottile/mkcert) command line tool.

1. Install [mkcert](https://github.com/FiloSottile/mkcert)
2. Run: `mkcert -install` to create your local CA (this only needs to be done once).
3. In your _nginx-proxy-base/certs/_ directory, create a new cert with: `mkcert yourlocaldomain.tld`. Make sure you replace "yourlocaldomain.tld" with the local domain name you will use.
4. After mkcert creates your certs, rename them:
    | FROM | TO |
    | - | - |
    | yourlocaldomain.tld-key.pem | yourlocaldomain.tld.key |
    | yourlocaldomain.tld.pem | yourlocaldomain.tld.crt |
5. If you use a TLD that is not _.localhost_ (see note below), you must add the domain name to your local machine's hosts file. This can be done on macOS with: `sudo vim /etc/hosts` and adding `127.0.0.1 yourlocaldomain.tld` to the list (make sure to replace _yourlocaldomain.tld_ with your actual domain name).

### A note on using the _.localhost_ TLD.

Since cURL version 7.85.0, domains ending in _.localhost_ can no longer resolve to an external IP
(see: [Curl localhost as a local host](https://daniel.haxx.se/blog/2021/05/31/curl-localhost-as-a-local-host/) and [Curl is not correctly resolving docker.localhost domain since version 7.85.0](https://github.com/curl/curl/issues/11104)).

This causes issues when using .localhost with some services, such as PHP and WordPress, because they use the cURL command to connect with other services. For example, WordPress uses it for its internal REST API and loopback connections.

For this reason, I recommend using a different TLD for your local domains, such as _.dockerhost_ or _.local_.

> Project using older versions of cURL, such as found in PHP versions before 8.1, will not have this issue.

## Using Git

By default, cloning a Git repository, which you did above in step one of the installation instructions, will clone the original .git directory. Any changes you push will attempt to go to the `git@github.com:jacobcassidy/docker-nginx-proxy-setup.git` repo, but you will get a permissions error, which is probably not what you want.

You have two options for using Git with your remote repository:

1. You can keep the cloned Git intact and change the connected remote repo.
2. You can remove the cloned Git repo and start with a clean repo.

### Option 1: Change the remote Git repo URL

1. Check which remote Git repo is currently in use with: `git remote -v`. This should output:
    ```shell
    origin  https://github.com/jacobcassidy/docker-nginx-proxy-setup.git (fetch)
    origin  https://github.com/jacobcassidy/docker-nginx-proxy-setup.git (push)
    ```
2. Change the remote repo used with: `git remote set-url origin <new-url>`. Make sure to replace `<new-url>` with your actual GitHub URL, such as `git@github.com:username/new-repository.git` or `https://github.com/username/new-repository.git`.
3. Run `git remote -v` to confirm your remote Git URL was correctly updated.

### Option 2: Start with a clean repo

1. In your _nginx-proxy-base_ directory, remove the cloned _.git_ directory with: `rm -rf .git`.
2. Initialize a new local Git repository with: `git init`.
3. Connect your remote GitHub repo as usual.

## Adding New Projects

When adding a new project to the nginx-proxy-network, keep the following in mind:

- You must include the nginx-proxy-network in the project. If your project uses a `compose.yaml` file, include the following network settings as a top-level element:
  ```yaml
  networks:
    default:
      name: nginx-proxy-network
      external: true
  ```
- Don't include ports for your services running on port 80 or 443 because the nginx-proxy service will take care of them. Other services, such as databases, should still use ports as usual.
- If you're using port 443 for the HTTPS protocol, you must create and add the TLS certs for your local domains in the `/nginx-proxy-base/certs` directory, and add the domain names in your local machine's hosts file (see "Using HTTPS with local domains" instructions above).

> Example projects that use the nginx-proxy-network include the [Docker WordPress Setup](https://github.com/jacobcassidy/docker-wordpress-setup) and the [Docker Nginx PHP-FPM Setup](https://github.com/jacobcassidy/docker-nginx-phpfpm-setup).

## Issues?

If you come across any issues, please feel free to report them [here](https://github.com/jacobcassidy/docker-nginx-proxy-setup/issues).
