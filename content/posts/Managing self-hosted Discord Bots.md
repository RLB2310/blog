+++
title = 'Managing Self Hosted Discord Bots'
date = 2024-06-28T09:48:06+10:00
+++

# Discord Bots

Discord Bots are an effective way to managing community servers through automation and interacting with API's. Realistically people aren't always available to moderate these things, so bots help.

I've taken a different approach to Discord Bots. Instead of moderating a server, the bots I have created respond to commands with processes on my [Raspberry Pi](https://rlb2310.github.io/posts/raspberry-pi-as-a-server/) where they are self-hosted.

This provides me with easy access to my server, giving me quick functions that I can use anywhere without having to SSH in, everytime I wanted to, say, turn on a service. Instead, I set a command for the bot to listen for in a text channel, and then respond accordingly when present.

The bot is also present in a server with my friends. We share a lot of documents over email, so I have also created another bot that I will cover elsewhere which handles file sharing.I also provide services for game servers for them. This also allows some of my less tech-savvy friends to interact with my server in a clean and simple interface.

![text](/assets/Images/Bots.png)

# The backend

To self-host discord bots, you need a device that is on as often as possible. This could be a cloud instance such as Linode, or just a spare computer.

Discord has it's own API that is built in multiple programming languages such as Javascript and Python. Because I already have experience in Python, I decided to use that.

The bulk of the code is setting up the bot alongside creating it in the [developer portal](https://discord.com/build/app-developers)

Starting with the imports:

``` import threading
import discord
import os
import random
import requests
import time
import asyncio
from discord.ext import tasks
import subprocess 
```

The Discord library is imported as the main API I'll be using.

Then I state the intents:

``` intents = discord.Intents.default()
intents.message_content = True

client = discord.Client(intents=intents)
```

The intents are defaults  for the Discord bot client. They will be the same across almost every bot with only the naming scheme differing.

The bot's "$help" command uses a dictionary of "command:descrption" pairs provided to the user in the Discord channel.


``` 
commands_list = [
    {'command': '$help', 'description': 'Show help message for document requests'},
    {'command': '$list', 'description': 'List available commands'},
    {'command': '$storage', 'description': 'Check the current storage status of the documents folder'},
    {'command': '$documents', 'description': 'List of documents'},
    {'command': '$add', 'description': 'Add document to backlog'},
    {'command': '$delete', 'description': 'Delete document from backlog'},
    {'command': '$scan', 'description': 'Scan documents for malware'},
    {'command': '$temp', 'description': 'Check the server temperature'},
]
```


![text](/assets/Images/Help.png)
