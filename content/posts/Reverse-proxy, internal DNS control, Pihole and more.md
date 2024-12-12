+++
title = 'Reverse Proxy, Internal DNS Control, Pihole and More'
date = 2024-07-02T11:05:31+10:00
+++
# Nginx for internal DNS control 

When interacting with my server, it's web services. and all the applications, it becomes tedious having to manually type in the IP address alongside the ports everytime. Luckily, reverse-proxies like Nginx are an exact solution to this.

A [reverse-proxy](https://en.wikipedia.org/wiki/Reverse_proxy) is usually a server that sits in front of web servers and forwards requests to them. They are usually confused with [load-balancers](https://en.wikipedia.org/wiki/Load_balancing_(computing)), and while they do have the ability to distribute tasks over different servers, they have the added benefit of providing ononymity by masking the origin IP.

![text](/assets/Images/Reverse.png)

For a selfhosting/homelab scenario, reverse-proxies are typically used as an authentication and security front-end when a port is forwarded and opened on a network. 

For my Use case, I will be forwarding both HTTP and HTTPS requests throughout my devices to eaily give myself access to all my services.  Nginx is pretty simple to use, having config files which allow you to set different forwarding requests. I currently have 3 Nginx configurations which I have set up (including SIlverbullet). I will be covering each one and it's uses in my network.

# HTTPS request to HTTP backend

One of the benefits of [Silverbullet](https://rlb2310.github.io/posts/note-taking-with-silverbullet/) as a note-taking application is the use of offline note storing with HTTPS.

Creating a HTTPS web service is simple enough, only requiring you to generate your own keys and certificates.

To do so i used something similar to the following:
```
openssl req -new > cert.csr
openssl rsa -in privkey.pem -out key.pem
openssl x509 -in cert.csr -out cert.pem -req -signkey key.pem -days 1001
cat key.pem>>cert.pem
```
This created all the certs and keys that I needed to start up an Nginx request, to forward that HTTPS request on the front-end, to the Silverbullet backend.


