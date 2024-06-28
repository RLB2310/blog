+++
title = 'Managing Self Hosted Discord Bots'
date = 2024-06-28T09:48:06+10:00
+++

# Discord Bots

Discord Bots are an effective way to managing community servers through automation and interacting with API's. Realistically people aren't always available to moderate these things, so bots help.

I've taken a different approach to Discord Bots. Instead of moderating a server, the bots I have created respond to commands with processes on my [Raspberry Pi](https://rlb2310.github.io/posts/raspberry-pi-as-a-server/) where they are self-hosted.

This provides me with easy access to my server, giving me quick functions that I can use anywhere without having to SSH in, everytime I wanted to, say, turn on a service. Instead, I set a command for the bot to listen for in a text channel, and then respond accordingly when present.

# The backend

To self-host discord bots, you need a device that is on as often as possible. This could be a cloud instance such as Linode, or just a spare computer.

Discord has it's own API that is built in multiple programming languages such as Javascript and Python. Because I already have experience in Python, I decided to use that.

The bulk of the code is setting up the bot alongside creating it in the [developer portal](https://discord.com/build/app-developers)

