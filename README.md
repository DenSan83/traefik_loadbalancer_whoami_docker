<p align="center">
<img src="https://d1q6f0aelx0por.cloudfront.net/product-logos/library-traefik-logo.png" alt="Traefik" title="Traefik" />
</p>

# Traefik loadBalancer & whoami in Docker

## Context

This project is a modification from [another one](https://github.com/DenSan83/traefik_whoami_docker) where the tasks are more simple: build a Traefik reverse proxy with two whoami micro servers, all managed by Docker compose. But at the beggining, when I was asked to do this project, I did not know what a "reverse-proxy" was. 

So I did my research on Google and got to the conclusion that it is the "oposite" of a proxy, meaning a computer could connect to a proxy who would then redirect the request to one or many servers without the first computer needing to know which one. 

My lack of understanding of the original request pushed me to understand "reverse-proxy" as something that needed a "load balancer" to distribute those charges. So I ended up adding it to the original request. 

I have a better understanding of a reverse proxy now, but I would not like to forget how to implement a load balancer, so I decided to create a git repo, so I could keep this to hands reach.

![Random reverse proxy image on the web](https://academy.avast.com/hs-fs/hubfs/New_Avast_Academy/What%20is%20reversed%20proxy/What_is_Reverse_Proxy_Server.png)

Needless to say, you're welcome to clone this if it is useful to you.


---

. **[Features](#features)** .
**[Build](#build)** .
**[Customize](#customize)** .
**[Turn off](#turn-off)** .
**[Extensibility](#extensibility)** .
**[Documentation](#documentation)** .
**[Original project](#original-project)** .

---

## Features

- Build and run with one command line
- Traefik as a reverse proxy
- Two whoami microservers with their unique subdomains
- One subdomain with a load balancer, able to call both of the whoami servers, or more if added.
- A password protected Dashboard to manage all services and routers


## Build

1. First, clone this repo on your folder:

```shell
git clone https://github.com/DenSan83/traefik_loadbalancer_whoami_docker.git
```

2. Next, assuming you already have Docker in your machine, open a CLI and type:

```shell
docker compose up -d
```

3. When the traefik and whoami images have been downloaded and run, you may try the following addresses on a browser or on your CLI with a curl command:

```shell
curl http://www1.localhost/
curl http://www2.localhost/
```
- They should both display different data from their seldom whoami servers

4. The load balancer can be seen from two different urls on a browser or on your CLI with a curl command:

```shell
curl http://www.localhost/
curl http://balancer.localhost/
```

5. You can also visit [http://traefik.localhost/](http://traefik.localhost/) to visit the dashbord and see what's going on. It will show you the current entrypoints, routers, services, middlewares and more. To connect to this dashboard, a user and a password will be needed. Use `root` as user and password for the first connection, but care to modify it for any real use.
![Web UI Providers](https://raw.githubusercontent.com/traefik/traefik/v2.5/docs/content/assets/img/webui-dashboard.png)
- Check out the [Customize](#customize) section to modify the files to suit your needs.

## Customize

### Change whoami subdomains

This can be easily done on the docker-compose.yml file, on lines 23 for the whoami1 service and 33 for the whoami2. Please make sure you dont delete the backticks (\`) arount the url and also keep the "whoami1" part so the loadbalancer can connect to it.

``` yaml
 - "traefik.http.routers.whoami1.rule=Host(`whoami1`) || Host(`<your_url>`)"
```

### Change the load balancer subdomain

This parameter can be found and changed on the traefik yaml file, on line 26. This example has already two routes you can modify. Please make sure you dont delete the backticks (\`) arount the url.

``` yaml
  rule: "Host(`<your_url1>`) || Host(`<your_url2>`)"
  # or you can also keep only one url :
  rule: "Host(`<your_url>`)"
 
```

### Change Traefik dashboard url

This parameter is also very simple to update, by modifying the api router rule on the traefik.yml file on line 21. Once again be careful with the backticks (\`) arount the url.

``` yaml
 rule: "Host(`<your_url>`)"
```

### Change Traefik dashboard user and password

There are two ways to do this. The first and more simple is via your (WSL) CLI.

First let's install the `htpasswd` utility to create this encrypted password. It is included in the `apache2-utils` package:

``` shell
 sudo apt-get install apache2-utils
```
Then generate the password with htpasswd. You will need to substitute your user name on `<user_name>` and your password on `<password>`:

``` shell
 htpasswd -nb <user_name> <password>
```
Copy the whole output (user, colons and hash) and replace the content on the traefik.yml file, on line 34, inside the double quotes. Feel free to delete or modify line 33 according to your needs.

``` yaml
- "root:$apr1$1kGQ8.wC$cfrIXE4XtaASRtnLDgi7N0"
```
Another way to get a user:password string done on `htpasswd` is by finding a service that can do this online, such as [this one](https://www.web2generators.com/apache-tools/htpasswd-generator). According to [Traeffik docs](https://doc.traefik.io/traefik/middlewares/http/basicauth/#general) you just need to make sure the hash is done using `MD5`, `SHA1`, or `BCrypt`. Then, you can use it to modify your traefik.yml file on line 34.

## Turn off

You can turn this app off with a single command line:
``` shell
docker compose down
```

## Extensibility

In order to add more whoami servers, each with a unique subdomain, please copy the last whoami service in the docker-compose file.

``` yaml
  # Second container of containous/whoami
  whoami2:
    image: containous/whoami
    container_name: whoami2
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami2.rule=Host(`www2.localhost`)"
    networks:
      traefik_network:
```
Feel free to edit the comment on the first line at will. Replace the `whoami2` name for a unique service name. Next, replace also the container_name field, preferably with the service name. Then, replace also the "whoami2" on "traefik.http.routers.**whoami2**.rule" by the new service name on the second label. Lastly, don't forget to replace the Host for a unique url, and as before, mind the backticks(\`) around.

Once your changes are all done, [turn off](#turn-off) the app and [build](#build) it again.

## Documentation

- [Docker](https://docs.docker.com/)
- [Traefik](https://doc.traefik.io/traefik/)
- [whoami](https://github.com/traefik/whoami)

## Original project
- [https://github.com/DenSan83/traefik_whoami_docker](https://github.com/DenSan83/traefik_whoami_docker)