+++
title = 'Note Taking With Silverbullet'
date = 2024-06-28T08:57:03+10:00
+++

# The problem

Majority of the notes I take for school are located under Microsoft's OneNote. Looking towards the future, I'd lose the education license after I had finished school, and considering I don't want to change note-taking apps after I've already got so much, I decided it would be best to change now.

# Obsidian

Obsidian is a closed-source markdown note application, available on all platforms. It seemed perfect. However the syncing of notes was a paid feature, and although using external applications such as [Syncthing]() or [Nextcloud]() was available, both had their own issues however. Syncthing was extremely inconsistent at file versioning, to the point where I would spent more time finding the correct version then actually writing notes. Nextcloud was really volatile, breaking at least once a week and forcing me to recreate the whole Docker container again.

# The final solution

**SIlverbullet**. It's perfect. A note-taking app which uses one server and progressive web-apps (PWA) for the clients. This reduced the different apps and clients I have to manage, by putting all the work onto the server. And that's not even the best part. Silverbullet supports two modes. Sync mode which periodically refreshes with the server, and Online mode which writes directly to disk. The sync mode supports offline note taking through HTTPS caching and web-browser local data storage. This means that as long as I haven't cleared my cache, I will always have a local version of the notes. 

# How is HTTPS enabled?

Enabling HTTPS on a HTTP self-hosted web-gui isn't too difficult. Using a reverse-proxy, you can forward a HTTPS request from a registered domain (either publicly or through the hosts file like I did) to a HTTP backend like the Silverbullet Web Client.

Using nginx, I've forwarded a HTTPS request for https://Silverbullet.local, to the :1313 port. I've also generated SSL cerificates to be able to use HTTPS. 

Here's The config file for /etc/nginx/sites-enabled/silverbullet.local: (local address used is 192.168.20.8)

```

# HTTP server block to redirect HTTP to HTTPS
server {
    listen 80;
    server_name silverbullet.local;

     location / {
         return 301 https://$host$request_uri;
     }
}

# HTTPS server block to handle secure traffic
server {
    listen 443 ssl;
    server_name silverbullet.local;

    ssl_certificate /etc/ssl/certs/selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/selfsigned.key;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    location / {
        proxy_pass http://192.168.20.8:3000;  # Proxy to the specific IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

Configuring individual sites with Nginx is pretty copy-and-paste, meaning that once you have set up one site, it's super easy to use a simialr configuration for the next. 

With Nginx successfully configured to forward the HTTPS request to the Silverbullet HTTP back-end, I have been able to use the app on all of my devices, syncing changes to my server when they are made.

