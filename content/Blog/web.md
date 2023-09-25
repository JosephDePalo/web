---
title: "Building & Hosting my Personal Website"
date: 2023-09-22T12:35:47-04:00
draft: false
---

# Introduction
As a college student looking to get their name out there, I decided to build a personal website. I wanted to have a place where I could showcase my projects, resume, and blog posts. I also wanted to have a place where I could experiment with new technologies and ideas. This post will go over the process of building and hosting my website.

# Building the Website
I did not intend to have any complex features on my website, so I decided to use a static site generator. Hugo is one such popular static site generator, and one of the most popular. It is written in Go, and is very fast. It also has a large community and a lot of themes to choose from.

To learn Hugo I enlisted the help of Youtube, particularly [Luke Smith's](https://www.youtube.com/@LukeSmithxyz) great [tutorial](https://www.youtube.com/watch?v=ZFL09qhKi5I). Hugo makes developing a website as simple as editing markdown files. For this site, I use Luke Smith's theme [lugo](https://github.com/LukeSmithxyz/lugo). 

# Hosting
## Docker & Nginx
For my own experience and digital sovereignty, I decided against paying for website hosting and instead opted to host this site myself on my [Raspberry Pi home server]({{< ref "/Blog/rpi4.md" >}} "Setting up a Raspberry Pi 4 Home Server").

First, I had to install Docker, which isn't as simple as `sudo apt install docker` on Debian 12. Following the instructions in the [Docker documentation](https://docs.docker.com/engine/install/debian/), I first set up Docker's apt repository and then ran `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`. I prefer using Docker Compose over bare Dockerfiles, so I also installed that with `sudo apt install docker-compose`.

At this point, getting everything running was as simple as cloning the [website repository](https://github.com/JosephDePalo/web), writing a docker-compose.yml file (see contents below), and running `docker-compose up -d` (the `-d` runs the container in a detached state so it doesn't occupy the terminal).

```yml
services:
  web:
    image: nginx
    volumes:
      - ./web/public:/usr/share/nginx/html
    ports:
      - "80:80"
```

## Putting it on the Web
### Purchasing & Using a Domain
Now that the website is set up, it's time to make it accessible to the web. First I had to register a domain name. This is as simple as finding a domain provider online (I used [Namecheap](https://www.namecheap.com/)) and purchasing a domain. I chose josephdepalo.com. Going into my domain's DNS settings, I added an A record pointing to my home IP address. This means that when someone types in josephdepalo.com, they will be directed to my home IP address where my web server is hosted.

### Setting up SSL with Certbot
Next was setting up SSL so my site can be accessed using HTTPS. For this I decided to use Certbot, a tool designed to generate free HTTPS certificates. While using Certbot is typically as simple as running `sudo certbot --nginx`, Docker complicated things. This turned out to be more complicated than I initially figured it would be, but I eventually found a [good guide](https://mindsers.blog/post/https-using-nginx-certbot-docker/). My docker-compose.yml file ended up looking like this:
  
```yml
  services:
  web:
    image: nginx
    volumes:
      - ./web/public:/usr/share/nginx/html:ro
      - ./nginx/conf:/etc/nginx/conf.d:ro
      - ./certbot/www:/var/www/certbot/:ro
      - ./certbot/conf/:/etc/nginx/ssl/:ro
    ports:
      - "80:80"
      - "443:443"
    restart: always
  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/www:/var/www/certbot/:rw
      - ./certbot/conf/:/etc/letsencrypt/:rw
```

I also now needed a nginx.conf file to specify the SSL settings. This is what mine looks like:

```conf
server {
    listen 443 default_server ssl http2;
    listen [::]:443 ssl http2;

    server_name josephdepalo.com;

    ssl_certificate /etc/nginx/ssl/live/josephdepalo.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/josephdepalo.com/privkey.pem;

    index index.html;

    location / {
        root /usr/share/nginx/html;
    }
}

server {
    listen 80;
    listen [::]:80;

    server_name josephdepalo.com;
    server_tokens off;

    index index.html;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        root /usr/share/nginx/html;
    }
}
```

# Conclusion
After a fair amount of troubleshooting, everything works. I now have a personal website that I can use to showcase my projects, post blog posts, and experiment with new technologies. Going forward I hope to continue to add to this website and use it as a place to learn and grow.
