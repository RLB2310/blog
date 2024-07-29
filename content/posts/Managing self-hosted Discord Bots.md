+++
title = 'Managing Self Hosted Discord Bots'
date = 2024-06-28T09:48:06+10:00
+++

# Discord Bots

[Discord Bots](https://discord.com/blog/how-to-use-discord-apps) are an effective way to managing community servers through automation and interacting with API's. Realistically people aren't always available to moderate these things, so bots help.

I've taken a different approach to Discord Bots. Instead of moderating a server, the bots I have created respond to commands with processes on my [Raspberry Pi](https://rlb2310.github.io/posts/raspberry-pi-as-a-server/) where they are self-hosted.

This provides me with easy access to my server, giving me quick functions that I can use anywhere without having to SSH in, everytime I wanted to, say, turn on a service. Instead, I set a command for the bot to listen for in a text channel, and then respond accordingly when present.

The bot is also present in a server with my friends. We share a lot of documents over email, so I have also created another bot that I will cover elsewhere which handles file sharing.I also provide services for game servers for them. This also allows some of my less tech-savvy friends to interact with my server in a clean and simple interface.

![text](/assets/Images/Bots.png)

# General Bot
## The backend

To self-host discord bots, you need a device that is on as often as possible. This could be a cloud instance such as [Linode](https://www.linode.com/), or just a spare computer.

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

## Storage state and space 

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

## ClamAV Integration

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

# Croc file sharing 

Finding a suitable solution to file sharing large documents was difficult to find. It's not that there's a lack of options, there's hundreds. The only problem is that they all have a file limit. Most online versions offer a free 500MB transfer maximum. Considering I was wanting to transfer a large Python program to a friend that was over that, and I wanted to end up transfering over 50GB of files, It wouldn't be applicable for me. 

Next I searched for self-hosted apps. [Wormhole]() is a popular app, which is open-source and allows unlimited file transfers as it uses public relays to conduct transfers. I used Wormhole for testing and noticed that it would often pause or cancel in the middle of a transfer. This was problematic, especially during large multi-gigabyte transfers. So Wormhole wasn't going to work.

[Croc](https://github.com/schollz/croc) is essentially the same as Wormhole except it's written in [Golang](https://go.dev/) and includes a couple more key features. Croc hashes files before the transfer starts, using encryption to protect your files. Similarly to Wormhole, Croc generates a code that another computer (local or outside of the network) can use to transfer the files/folder. 

A key difference that Croc includes that makes it applicable in my situation is the ability to continue/resume downloads that have cancelled. It requires you to create a code for the same files/folder and for the other computer to continue the transfer. As to how it continues and understands the files that needs to be resumed, I am unsure of. It lacks extensive documentation, however, if I do so much as to change anythng in the files (properties from name, bytes within it) the resume won't pick up. This most probably means that it compares the hashes of the files to determine what files are the same.

So Croc works perfectly. With speeds in-line with my upload trasnfer speeds, file transfers are relatively fast. Now I had to make it as automated as possible.

Croc required a large amount of work from me. I can't always provide the codes for each document that I have, sometimes my friends need notes for school late at night. So working it into a Discord bot, similar to what was done above was a good solution.

## Croc and Discord
   
The approach I've taken is very similar to the general bot shown previously. The Croc bot is set to listen to a specific channel to reduce the amount of clutter in the general channel. 

Once the bot recieves the command alongisde the document name as an argument, it then conducts a server-side check for the name, and if found, it will generate a croc code with the subprocess library.

![text](/assets/Images/croc_example.png)


This removes the need for me to interact with my server at all when another person is conducting a transfer. For updates, changes, and when it occasionally breaks I do need to intervene though.


*Overall* I'd say that investing time into Discord bots has made life a lot easier when it comes to iteracting with my homelab, as well as providing automnated services for my friends and family. The degree of autonomy is only so high, and to be honest I think majority of the hard work and heavy lifting is taken care of.
