---
title: Use a Custom Domain for Your Local Docker Containers with `caddy-docker-proxy`
date: 2025-05-29
tags: [docker, reverse-proxy ]
---

I recently discovered that using [caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) is an excellent solution for setting up local domains for my Docker containers. It has become my go-to reverse proxy setup for Docker containers. 

## Nginx Proxy Manager VS Caddy Docker Proxy

Before this, I was using [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager), which works great if you're using verified public domains. But for local development, it’s needs extra work, which I don't have time for. While I did consider using self-signed SSL certificates, the setup process seemed more trouble than it was worth.

Then I remembered that **Caddy supports self-signed certificates out of the box**. After some digging, I found [caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) — and it worked like a charm.


## Step 1: Point Your Local Domain to 127.0.0.1

First, I added an entry to my `/etc/hosts` file to point a custom domain to my local machine:

```bash
127.0.0.1 ynrfin.local
```


## Step 2: Create a Docker Network

Next, I created a dedicated Docker network called `caddy`, which both the proxy and my services will share:

```bash
docker network create caddy
```


## Step 3: Run the `caddy-docker-proxy` Container

Here’s the basic `docker-compose.yml` config I use:

```yaml
version: "3.7"
services:
  caddy:
    image: lucaslorentz/caddy-docker-proxy:ci-alpine
    ports:
      - 80:80
      - 443:443
    environment:
      - CADDY_INGRESS_NETWORKS=caddy
    networks:
      - caddy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - caddy_data:/data
    restart: unless-stopped

networks:
  caddy:
    external: true

volumes:
  caddy_data: {}
```

Important notes:
- Ports **80 (HTTP)** and **443 (HTTPS)** must be exposed. This is the entry point for every request to other containers.
- The proxy must be connected to the external `caddy` network. 


## Step 4: Configure Another Container (e.g. WordPress)

Here’s how I configured a WordPress container to work with the proxy:

```yaml
services:
  wordpress:
    image: wordpress:latest
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - db
    networks:
      - wp_internal
      - caddy
    labels:
      caddy: ynrfin.local
      caddy.tls: internal
      caddy.reverse_proxy: "{{upstreams 80}}"
  db:
    ...
networks:
  wp_internal:
    internal: true
  caddy:
    external: true
```

Key points:
- The `labels` section inside the `wordpress` service defines how caddy-docker-proxy handles routing.
- I didn’t expose WordPress directly, caddy-docker-proxy will route the request and response for me.
- caddy-docker-proxy inspects other containers via Docker and looks for `caddy` labels to know where to route requests.


## Handling Multiple Ports (e.g. Laravel + Vite)

For Laravel projects using both port `80` (PHP server) and `5173` (Vite dev server), here’s how I handle it:

```yaml
labels:
  caddy_0: botlog.local
  caddy_0.reverse_proxy: "{{upstreams 80}}"
  caddy_0.tls: internal
  caddy_1: botlog.local:5173
  caddy_1.reverse_proxy: "{{upstreams 5173}}"
  caddy_1.tls: internal
```

Explanation:
- `caddy_0` handles HTTP (port 80)
- `caddy_1` handles Vite assets (port 5173)


## Final Thoughts

Using `caddy-docker-proxy` has made my local development workflow much smoother. I no longer have to remember which app use what port, as I can use more humane address using domain 

If you're dealing with local Docker containers and want proper domains and HTTPS — especially for multiple projects setups — I highly recommend giving this a try.

