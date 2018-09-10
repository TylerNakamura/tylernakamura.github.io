---
layout: post
title: My Seedbox Configuration and Guide
date: 2016-05-03 12:41
comments: false
categories: seedbox vps rtorrent screen
---
Below is a guide and a tale of my efforts in creating an seedbox:

### Selecting a Virtual Private Server Service

I chose to go with a VPS from [Dediseedbox.com](http://dediseedbox.com/). The fact that they are built towards seeding made me choose them as my provider. Compare [their rates](http://dediseedbox.com/vps.html) to another popular provider like [Digital Ocean](https://www.digitalocean.com/pricing/). Having servers located outside of the US can also be advantageous for a variety of other reasons.

I was pretty satisfied with the speeds, especially considering the location. For myself, the speeds exceed sufficiency.

<img src="/images/speedtests.png"/>

### Setting up SSH Keys

SSH keys can allow a client to SSH into a VPS without the use of a password. When configuring a VPS, you may need to log into your server often.
Setting up keys can help speed up this process. SSH keys can also enhance security if you choose to still use a password.
SSH keys aren't required but can be very convenient.
For more information, [Digital Ocean has an excellent basic guide for setting up SSH keys.](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

For maximum laziness and brownie points, look into creating an object in your ~/.ssh/config that defines a certain ssh connection so that you can connect like so:

```bash
ssh france
```

my bash alias automatically logs me into the VPS; all done securely without the use of a password.

For more info on bash aliases, [see here](https://www.digitalocean.com/community/tutorials/an-introduction-to-useful-bash-aliases-and-functions).

### Configuring rTorrent
"rTorrent is a text-based ncurses BitTorrent client written in C++, based on the libTorrent libraries for Unix, whose author's goal is a focus on high performance and good code." [(source)](https://en.wikipedia.org/wiki/RTorrent).
I have made rTorrent my BitTorrent client of choice as it functions via the CLI.

Much of the documentation that you need in configuring rTorrent can be found at the [Arch Linux wiki](https://wiki.archlinux.org/index.php/RTorrent).
However, this segment will mostly pertain to what I consider to be the essentials in my rTorrent configuration.

My apt-get installation for didn't seem to include a copy of the **rtorrent.rc** config file (or at least I couldn't find it). To download the rTorrent config file from the command line:

```bash
wget https://raw.githubusercontent.com/rakshasa/rtorrent/master/doc/rtorrent.rc
mv rtorrent.rc ~/.rtorrent.rc
```
Save the downloaded config file in the home directory of the user that will be running rTorrent.

Many of the settings aren't required for basic rTorrent usage, but here are the ones that I found essential:

Changing the directory variable will specify where downloaded file will be placed:

```bash
# Default directory to save the downloaded torrents.
directory = /home/user/path/to/folder
```

Specifying a session directory will allow rtorrent to save sessions:

```bash
# Default session directory. Make sure you don't run multiple instance
# of rtorrent using the same session directory. Perhaps using a
# relative path?
session = /home/user/path/to/folder
```

Perhaps the most important setting for automation, a watch directory that will be monitored for .torrent files.
When a .torrent file is found in this directory, rtorrent (if running) will take care of the rest and place the respective files in the sessions folder and the downloads folder.

```bash
# Watch a directory for new torrents, and stop those that have been deleted
schedule = watch_directory,5,5,load.start=/home/user/path/to/watch/directory/*.torrent
schedule = untied_directory,5,5,stop_untied=
```

There are many other options for customization, be sure to check out the Arch Wiki or [this blog post](https://harbhag.wordpress.com/2010/06/30/tutorial-using-rtorrent-on-linux-like-a-pro/) for more information about rTorrent config.

### Running rTorrent in the Background with Screen
 [I found this solution on a forum](http://www.linuxquestions.org/questions/linux-general-1/problem-using-screen-cannot-open-your-terminal-'-dev-pts-0'-please-check-338313/) that describes how to run rTorrent in the background.
 Opening rTorrent via **screen** will allow you to detach and reattach from a running screen:

```bash
screen -S "rtorrent" rtorrent
```

Pressing ctr+a, then ctrl+d will detach from the screen while leaving the process running.

To list all of the screens for a user, run the following command as that user[(source)](http://stackoverflow.com/questions/537942/how-to-list-running-screen-sessions):

```bash
screen -ls
```

To reattach to a screen with the name of rtorrent:

```bash
screen -r rtorrent
```

### Converting Magnet Links to .torrent Files
[In 2012, The Pirate Bay no longer offered torrent files but magnet links instead](http://www.webcitation.org/6BWzbw7JF). This decision provided another layer of abstraction in the hope that it would mitigate some of the litigious efforts to banish The Pirate Bay.
It also reduced the amount of bandwidth necessary to keep the site running through a proxy.
As a user, all you really have to know is that a magnet link stores meta data about a torrent file and it requires some minor conversion in order to retrieve the .torrent file that you need.
There are sites that allow you to do this conversion from a browser:

<a src="http://torrent-to-magnet.com/"><img src="/images/torrent-magnet-site-screenshot.png"></a>

However, I need magnet conversion from the command line, I [found a nifty script](http://snarvaez.com.ar/libertad/index.php/2013/05/10/download-magnet-links-with-rtorrent-from-command-line/) that helped me create this.
The below script will convert a magnet link to a .torrent and copy it to a remote host.
Be sure to change the target directories in the scp function to your personal information.

```bash
#!/bin/bash

# converts the magnet link into a .torrent file in the current directory
[[ "$1" =~ xt=urn:btih:([^&/]+) ]] || exit;
echo "d10:magnet-uri${#1}:${1}e" > "meta-${BASH_REMATCH[1]}.torrent"

# securely copies the torrent file to your remote server, be sure to change path
scp "meta-${BASH_REMATCH[1]}.torrent" user@remote_host:/path/to/rtorrent-watch-directory

# if you don't want to keep the .torrent file just call rm on it
#rm "meta-${BASH_REMATCH[1]}.torrent"
```

The script usage looks like this:

```bash
./script "magnet link here"
```

Note that the magnet link needs to be enclosed by quotation marks.

## More Resources
- [Guide for Optimizing Torrent Speed](https://torrentfreak.com/optimize-your-bittorrent-download-speed/)
- [Digital Ocean - How to Set up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
- [Digital Ocean - Bash Alias Tut](https://www.digitalocean.com/community/tutorials/an-introduction-to-useful-bash-aliases-and-functions)
- [Download Magnet Links with rTorrent From Command Line](http://snarvaez.com.ar/libertad/index.php/2013/05/10/download-magnet-links-with-rtorrent-from-command-line/)
