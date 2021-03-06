# letsencrypt-nginx-proxy
letsencrypt-nginx-proxy is based on [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy/). It sets up a container running nginx and [docker-gen](https://github.com/jwilder/docker-gen).
See [Automated Nginx Reverse Proxy for Docker](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/) for why you might want to use this.
In addition to the functionality that [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy/) offers (reverse proxy configs for nginx and reloads nginx when containers are started and stopped) we use docker-gen to generate a SSL certificate from letsencrypt to secure the domain.

### Usage
If you want to run it without SSL support, please have a look at the page for the  [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy/). That's what you're actually looking for.

To run it:
    `docker-compose up -d`

That's about it already. If you want to run it without `docker-compose` it would like this:

   `docker run -d --name nginx-proxy -v /var/run/docker.sock:/tmp/docker.sock:ro -e LETSENCRYPT_EMAIL=<your_email@domain.de> --restart=always eforce21/letsencrypt-nginx-proxy`

### Configuration
You can configure the email address that should be used for certificate generation with letsencrypt with the environment variable `LETSENCRYPT_EMAIL`. If you do not set it, the email address will defaul to `info@VIRTUAL_HOST`.

#### Opt Out of Certificate Generation (default)
By default this proxy will attempt to obtain a certificate for all containers that have a VIRTUAL_HOST environment variable. If you don't want SSL support for a certain container you can add a label to prevent certificate generation: `letsencrypt.nocert`. The value you assign is not checked right now. Only the existence of the label is enough to exclude for certificate generation. That's how it would look like with a run command: `docker run -tid --label letsencrypt.nocert=true -e VIRTUAL_HOST=<your_domain> ubuntu`

#### Opt In to Certificate Generation
Alternatively, you can configure the proxy to never generate a certificate for a container unless it defines a label to request certificate generation.  To do this you will need to start the container with the environment variable LETSENCRYPT_OPT_IN. The run command would look like this:

  `docker run -d --name nginx-proxy -v /var/run/docker.sock:/tmp/docker.sock:ro -e LETSENCRYPT_EMAIL=<your_email@domain.de> -e LETSENCRYPT_OPT_IN=true --restart=always eforce21/letsencrypt-nginx-proxy`

When the `LETSENCRYPT_OPT_IN` variable is present the proxy will look for containers with the label `letsencrypt.cert` and will generate certificate requests for them if they also have the `VIRTUAL_HOST` variable set. The run command would look like this: `docker run -tid --label letsencrypt.cert=true -e VIRTUAL_HOST=<your_domain> ubuntu`

#### Other Configuration
If there's anything else you want to configure. Please also have a look at [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy/). There you'll find more beautiful documentation on how to do more magic with this reverse proxy.

### How does it work?
We use [Let's Encrypt](https://letsencrypt.org/) to generate the SSL certificates. Those certificates are free and expire every 3 months.
We use [docker-gen](https://github.com/jwilder/docker-gen) to watch for starting containers and generate a shell-script that will run [Let's Encrypt](https://letsencrypt.org/). This will give you a SSL certificate in a matter of a couple of seconds. (So please don't worry when the certificate won't show up right after you start the container for the first time!). We use the `--keep-until-expiring` flag so you hopefully don't run into [beta restrictions](https://community.letsencrypt.org/t/public-beta-rate-limits/4772). That means the certificate will be renewed if it expires in 10 or less days automatically on container (re)start.
Additional we have `cron` installed in the container to check regularly that your SSL certificates don't expire as you might not (re)start your containers every 3 months. That check will be performed at 10am. If you want to change that, just change it in the `cronfile`.

### Docker Tags
`latest` is always taken from develop branch. Please do NOT consider it production ready. Use the versioned tags instead for production please!
