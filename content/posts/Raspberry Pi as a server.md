+++
title = 'Raspberry Pi as a Server'
date = 2024-06-26T16:32:01+10:00
+++

# Setting up a self-hosted home server

Why settle for something done poorly when you can do it better yourself? That's the approach I took with creating a general, all-in-one media server / NAS. The main concern was relying on external, third-party software and services to conduct daily tech and media activities. The solution? Do it yourself. I've done exactly that, setting up a Raspberry Pi that I can tinker with and use daily, helping not only me but also my friends and family.

Using a [Raspberry Pi](https://www.raspberrypi.com/) 4 (4GB), an external hard-drive, and SD card, I've managed to get a mini PC running Raspberry Pi OS Lite for under $300. Sounds like a lot, but once you see what it can do and the money it's saving, it's arguably worth it.

# Why a Raspbery Pi?
Mini PC's in Australia (and mobile devices in general) are expensive, meaning that sticking to the popular, tried and true Raspberry Pi ARM mini PC. Running a device 24/7 also raises concerns about power consumption. Runing my desktop PC idling at 80 Watts obviously isn't ideal. So a the Raspberry Pi is an obvious choice, idling at only 3-4 Watts on average with background tasks.

# Uses 

Having 4GB of RAM leaves a bit of room for Docker containers and web services, but obviously doesn't allow for extensive simultaneous hardware-acceleration or any gaming. Plenty of memory for my use case though, measuring the current usage with all the background processes, it uses about 2.9 out of the 4GB of memory available, leaving heaps of breathing room.

Currently some of the uses that the provides for me is an extensive amount of media management, from Plex and Jellyfin, providing access to movies and tv shows personally sourced from either blu-ray or on disc, to a music player backend using Navidrome which let's me source my own music and access it anywhere.

Majority of the services are deployed through [Docker](https://www.docker.com/) (even though I prefer barebones) because of the automatic image updates through [Watchtower](https://github.com/containrrr/watchtower). Some services I had trouble with accessing specific folders mounted on the external drive through Docker, alognside some permission conflicts. This was solved by using the barebones version.

Managing all the docker containers was complete through 


![text]()
