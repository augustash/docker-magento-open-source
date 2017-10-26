# Magento Open Source Docker Stack

![https://www.augustash.com](http://augustash.s3.amazonaws.com/logos/ash-inline-color-500.png)

**This container stack is not currently aimed at public consumption. It exists as a starting point for August Ash development.**

## Usage

This repository provides an easy way to spin up a Magento cluster of Docker containers. It's ideal for getting a Magento sandbox up and running quickly or as a starting point for developing a new Magento project locally.

### Sandbox

To create a Magento sandbox environment, you'll need first install Magento and then start the cluster:

```bash
docker-compose -f docker-compose.yml -f docker-compose.admin.yml \
    run --rm admin installer
docker-compose -f docker-compose.yml -f docker-compose.admin.yml \
    run --rm admin magento deploy:mode:set production
docker-compose -f docker-compose.yml -f docker-compose.admin.yml \
    run --rm admin magento setup:static-content:deploy
```

```bash
docker-compose up -d
```

### Development Environment

You can include the `dev` YAML file to create a Magento development environment. This will expose additional ports, and mount a host directory of `src/` into your containers. It follows a similar procedure as outlined above:

```bash
mkdir src/
```

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml \
    run --rm admin installer
docker-compose -f docker-compose.yml -f docker-compose.dev.yml \
    run --rm admin magento deploy:mode:set developer
docker-compose -f docker-compose.yml -f docker-compose.dev.yml \
    run --rm admin magento dev:urn-catalog:generate .idea/misc.xml
docker-compose -f docker-compose.yml -f docker-compose.dev.yml \
    run --rm admin magento setup:upgrade
```

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

## Local DNS

To run a cluster like this on your local machine, I am assuming you have a local DNS service or `hosts` file setup to respond to your requests.

I would recommend running `dnsmasq` as a container and routing all `.dev` domains to your local machine. Here's how I do it.

Create the file `/etc/resolver/dev` with the following content:

```bash
nameserver 127.0.0.1
port 53535
domain dev
search_order 1
```

I set it up this way so we don't need to run the container with elevated priviledges. The following with keep a DNS container running permanently until manually stopped:

```bash
docker run -d --name "dnsmasq" --restart always -p 53535:53/tcp -p 53535:53/udp --cap-add=NET_ADMIN andyshinn/dnsmasq:2.78 --address=/dev/127.0.0.1
```
