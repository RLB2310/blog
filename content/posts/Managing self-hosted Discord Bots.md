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
I then create an elif statement to check for the prefix of the message being "$help"
```
elif message.content.startswith('$help'):
        command_descriptions = [f"{cmd['command']}: {cmd['description']}" for cmd in commands_list]
        await message.channel.send("List of available commands:\n```" + '\n'.join(command_descriptions) + "```")

```

Now, whenever someone messages "$help", the bot responds accordingly.

![text](/assets/Images/Help.png)


This is only one example of the bot performing simple tasks. There's a multitude of commands such as getting the temperature of the server, getting the storage status, or listing the documents available.

# Storage state and space 

Determining the state of the storage on my server is helpful to identifying whether I need to make space and whether the server is downloading something at that point in time.

Finding the amount of space is simple enough. using a subprocess call using Python's Subprocess library allows me to send commands in the program as if it was a bash terminal. 

To determine the state of storage (increasing) a measurement is taken between two different time periods, and a comparison is made. An initial message is sent stating the storage size, and then a follow-up if it increases.

The bot first checks the message for the "$storage" command:

```
elif message.content.startswith("$storage"):
```

Following this, the storage space free is calculated by calling a command called "get_directory_size":
```
        initial_size = get_directory_size(Documents_folder)
```

The command starts with a variable (total_size) equal to 0, and loops through each file in the folder (recursively) and adds the size to the variable until it has no more files to go over. Then it returns the total size

```
def get_directory_size(directory):
   total_size = 0
   with os.scandir(directory) as it:
        for entry in it:
            if entry.is_file():
                total_size += entry.stat().st_size
            elif entry.is_dir():
                total_size += get_directory_size(entry.path)
   return total_size
```

Going back to the message, using the total size, it sends a message to the channel regarding the size, then waits 20 seconds before repeating, to then compare the two sizes and determine the activity of the server as increasing or not.

```
        gb_init = initial_size / 1024**3
        await message.channel.send(f"```Collection is currently sized at {gb_init:.2f} GB's```")
        await message.channel.send(f"Calculating current status...")
        time.sleep(20)
        current_size = get_directory_size(Documents_folder)
        if current_size > initial_size:
            await message.channel.send(f"```And the size is increasing, now at {current_size / 1024**3:.2f} GB's...```")
        else:
            pass
```

# ClamAV Integration

[ClamAV](https://www.clamav.net/) is a popular open-source antivirus engine for detecting trojans, viruses, malware & other malicious threats. Although it's not necessarily required and can be considered quite "extra", it's always better to be safe than sorry. I have a bunch of open-source applications from outside of the Debian repositories from Github and the likes, so having that extra protection is always good, especially when I can't scan through the whole project that I'm using.

So ClamAV isn't difficult to set up. Simply downloading from the Debian repositories and updating the virus database, ClamAV let's you conduct quick folder scans, as well as seperate applications for real-time protection. 

Creating a command for Discord wasn't difficult either, as ClamAV is already a command line program. 

Simply by checking the message for the scan command, we can then run an asynchronous call for the ClamAV command line program and then get the results from the directory and send it as a message.
 
```
elif message.content.startswith('$scan'):
        await message.channel.send("Scanning documents for malware... This may take up to 10 mins")
        scan_result = await run_clamav_scan('/media/root/EXTHDD/Documents')
        await message.channel.send(f'ClamAV Scan Results for Documents:\n```{scan_result}```')
```

