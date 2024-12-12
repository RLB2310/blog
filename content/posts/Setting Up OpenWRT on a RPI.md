+++
title = 'Setting Up OpenWRT on a RPI'
date = 2024-11-18T16:26:52+11:00
+++

# Raspberry Pi, as a router

I've decided that the Raspberry Pi 4 isn't holding up as a NAS, Music server, Plex Serve, etc. (understandably), so i'm turning it into a router because why not? And i'd like to learn about VLAN's and networking in general. 

## Firmware / OS Choice

Unfortunately, the more popular custom router firmware, PFsense (or Opnsense, even) don't support ARM architecture, so OpenWRT is the next choice. 

## The Setup

Jumping straight into it, I've flashed the Raspberry Pi 4 image onto an SD card, and expanded the filesystem, although i'm still unsure about the actual size of the filesystem, and started up the Raspberry Pi. While it does take a bit of trial and error to find the Web GUI address that it uses, using network scan commands such as "arp" helps with determining whether i'm on the correct subnet or if I need to adjust my configurations. 

The configuration for setting up the Pi as an access point and providing devices withh internet is easy enough however, actually giving an internet connection to the Pi at the same time doesn't seem to be working for me. Over the next couple of weeks, as I build the setup, I will have to play around with the VLAN's, to make sure I can route the traffic correctly.

## VLAN's

For a brief overview of how it's going to be setup, I'll have two VLAN's (aside from the default VLAN ID of 1) which will be VLAN 10 (WAN) and VLAN 20 (LAN). My goal is to have one ethernet port on my Raspberry Pi being used to route the traffic on the network, through my managed switch. I still have a bit to learn when it comes to VLAN's, however I have the basis of tagging and untagged ports figured out. 

  
