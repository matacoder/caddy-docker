# Caddy-Docker: Infrastructure as a Service

In this repository I would like to share with you a way to deploy new projects to you Docker Instance in favorite cloud.

I like Caddy a lot, but everytime I need to add new rule I had to ssh to my server, change configuration and do `caddy reload`. Never again!

Introducing Caddy-Docker!

# Run Caddy in Docker Container

Well, that is not hard to tune caddy for project if you have one and only one. But when you want to manage several projects and their static files with one instance of caddy you must follow these rules:

## 1. Create a docker compose file for caddy:

```yaml
version: '3.9'

volumes:
  caddy_data:
  caddy_config:

services:
  caddy:
    image: caddy:2
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      # Caddy specific urls
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
```

That's the basic first config. Just deploy it.

## 2. Create your project docker-compose and add caddy network to it:

```yaml
networks:
  default:
    external:
      name: caddy-docker_default
```
Now we can access these containers from caddy. Nice!

## 3. Connect volumes you want to serve with caddy to caddy docker compose.

Assume you have project in folder `django-docker` that creates volume `static`. That means the actual volume name would be `django-docker_static`.

If you struggle to find proper name just `docker volume ls` on your server to find it out.

Add volumes to volume section:

```yaml
volumes:
  caddy_data:
  caddy_config:
  # Your project static or media files
  django-docker_media:
    external: true  # Means here this volume created outside this docker compose
  django-docker_static:
    external: true
 ```
 
 `external: true` as you may guess external here means that caddy's docker compose must not create them.
 
 ## 4. Mount volumes to caddy container

```yaml
services:
  caddy:
    image: caddy:2
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      # Caddy specific urls
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
      # Your project static or media files
      - django-docker_media:/django-docker/media
      - django-docker_static:/django-docker/static
 ```
 
 Now Caddy can access them
 
 ## 5. Write `Caddyfile` to redirect logic and serve files

As example here we you Django static and media files:

 ```
http://127.0.0.1 {
    handle /static/* {
		root * /django-docker
        file_server
    }
      handle /media/* {
      root * /django-docker
          file_server
    }
    handle {
          reverse_proxy django:8000
    }
    log
}
```
 
 Instead of ` http://127.0.0.1` you must write your domain, like `matakov.com` and caddy will issue SSL automatically.
 
 If you want to test it with localhost or without SSL just write `http://` before host.
 
 And obviously you must use your name of django service in `reverse_proxy django:8000`, port likely to be `8000` as default to gunicorn.
 
 # Serve Django static and media files with Caddy
 
 Just remember you must add these lines to `settings.py`:
 
 ```python
STATIC_URL = "/static/"
STATIC_ROOT = os.path.join(BASE_DIR, "static")

MEDIA_URL = "/media/"
MEDIA_ROOT = os.path.join(BASE_DIR, "media")
```
 
 
 
 
 
 
