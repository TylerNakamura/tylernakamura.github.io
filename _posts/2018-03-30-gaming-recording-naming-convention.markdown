---
layout: post
title: "GRNC1 - A Gaming Recording Naming Convention"
date: 2018-03-30
comments: false
published: True
categories: gaming recording naming convention
---

GRNC1 stands for *Gaming Recording Naming Convention Version 1*.
GRNC1 outlines a method for naming gaming recording video files.
This was created to support my personal needs in naming all of my video files.
Below outlines the protocol and some of my thought processes.

---

|  Field| Description | Examples |
|---|---|---|
|Date  | Limited [ISO8601 Timestamp](https://en.wikipedia.org/wiki/ISO_8601)|`2018` `2018-01` `2018-12-29` `2018-03-30T15`  |
||||
|Game|Name of Game Played | `Runescape` `Fornite` `FN` `SC2` `WoW` |
||||
|Players| Comma separated list of player names| `tyler` `todd, billy joel, sam` |
||||
|Description| Description of the event| `fun` `awesome fun times` `that one time that John farted` `ranked games`|
||||
|Rewatch Value|  A number from 0 to 9 describing your desire to rewatch later. 9 is most rewatch worthy | `0` `5` `9`|
||||
|Full or Partial Recording| Distinguish between full recordings and clip, f is a full recording, c is a clip| `f` `c`  |
||||

### Rules:
- Use a single `_`(underscore) to delimit each field.
- Empty fields are acceptable, but still use the delimiter.
- Spaces are tolerated, they are _not_ being used to delimit anything, so go crazy with them.
- While ISO8601 is required, not all characters in the standard are allowed by all file systems. So we have to work with a limited version of ISO8601. Check your file system constraints.

Examples:
```
2018-01-01_HOTS_timothy, sally, janet jackson_lots of fun with lost vikings_5_c.mkv
2018-02-02_Halo5_tyler_just solo play with battle rifle_0_f.mkv

# missing fields are ok
2018_WoW__fun_5_f.mkv
2018__tyler_fun_5_f.mkv
2018_wow_tyler__5_f.mkv
```

---

## Motivation

Over the last 3 years, I have been recording all of my gaming sessions with friends.
I'm not sure why I started this, but I feel that one day these old videos may be of use.
It's possible that when my brain is too slow, and my fingers too feeble, that I may want to go back to the glory days and rewatch hours of gaming footage.
I don't know, but its a hobby of mine to collect these videos and categorize them.

Now that I have accumulated hundreds of videos and TBs worth of footage, with many different games, and many different people, the demand for a proper naming convention is greater than ever.
By saving things like the game that was played and who it was played with allows me to easily search through my archives.
Gaming footage of my session with Bob will not be relevant or interesting to Alice, and vice versa.

I have plans on creating some kind of interface or search engine for this archive.
It also makes it easier for AI/ML to process these files.
There are many reasons on why you might want to name your files properly.

## Database Instead of Overloading File Names

I think an argument could be made that as long as each file name is unique, I could use a proper database to store data about each of the files.
This would allow for proper data storage and a larger amount of information stored for each video.
While I considered this, I think that there wasn't enough information that I could store for each video for a database to be worth it.
I chose instead to go with the simpler approach of storing all of this information in the file name itself.
This way, I don't have an additional data set to worry about.
This makes both upkeep and migration significantly easier.


## Character Limits in File Names across File Systems
Cross platform viability is important to me.
Operating systems evolve, as does their file systems.
If I wanted to move the storage of these videos onto a different filesystem, I should not be hindered by the size of the file name.
Luckily, it seems like these limits are largely the same for the file systems I considered.

Here are the file systems that I wanted to account for along with their file name size limits:

|Filesystem   | Maximum File Name Limit  |
|---|---|
|NTFS|  255 Characters|
| UFS1 | 255 Characters |
| UFS2 | 255 Characters |
| Ext1 | 255 Bytes |
| Ext2 | 255 Bytes |
| Ext3 | 255 Bytes |
| Ext4 | 255 Bytes |
|ZFS  | 255 Bytes |
| Btrfs| 255 Bytes |

Luckily, it seems that they are pretty similar. Otherwise, I probably would have chosen the smallest or most constraining limit.
If you are curious for more, check out this [Wikipedia page on a comparison of file systems](https://en.wikipedia.org/wiki/Comparison_of_file_systems).

## Choosing a Field Delimiter
Choosing the delimiter was the most challenging aspect of coming up with a naming convention.
While many delimiters work fine, I was especially picky about achieving the ideal one.
An obvious candidate is the `-`.
This is similar to what Plex uses in their chosen Television naming convention, it looks like this:
```
# Plex TV Show Naming Format
The Big Bang Theory - s04e10 - The Hot Troll Deviation.mkv
Friends - s01e01.mkv
```

While this delimiter is great for these kinds of files, it doesn't make sense to do this for gaming recordings.
This is mainly because the we require the ISO8601 dating standard in GRNC1.
ISO8601 makes use of `-`s as its own delimiter.
Plex doesn't incorporate that, so we have to find our own delimiter that transcends this.
For example, imagine if GRNC1 used `-` as a delimiter:
```
2018-01-01 - FN -Tyler, Bean - desc - 9 - c.mp4
```
This file would really throw off parsing as the parser wouldn't be able to differentiate what the delimiter is delimitting.
It would be cumbersome to distinguish ISO8601 dashes from GRNC1 dashes.
A `-` will not work well for GRNC1.

This means that we have to find a delimiter that satisfies these constraints:
1. Won't ever be a desired character in any of the fields
2. Is an acceptable character in _my preferred_ filesystems (FAT, NTFS, EXT3/4, ZFS)
3. Visually appealing
4. Easy to produce
5. Plays nicely with Bash

There is a lot to think about here, and I recommend reading [this post](https://stackoverflow.com/questions/1976007/what-characters-are-forbidden-in-windows-and-linux-directory-names) if you are interested in learning more.
But to keep it simple, let's just say we _cannot_ use:
```txt
Linux/Unix:

/ (forward slash)

Windows:

< (less than)
> (greater than)
: (colon - sometimes works, but is actually NTFS Alternate Data Streams)
" (double quote)
/ (forward slash)
\ (backslash)
| (vertical bar or pipe)
? (question mark)
* (asterisk)
```

So what does that leave us? Well to satisfy constraint 4 (easy to produce), I preferably want to use something that is easy to find on a keyboard.
This leaves us with:
```
  (space)
` (backtick)
~ (tilde)
' (single quote)
@ (at)
# (octothorp)
$ (dollar sign)
% (percent)
^ (caret)
& (ampersand)
( (left parenthesis)
) (right parenthesis)
- (dash)
_ (underscore)
= (equals)
+ (plus)
[ (left square bracket)
] (right square bracket)
{ (left curly bracket)
} (right curly bracket)
```

It's a matter of personal preference at this point.
I strongly considered a right/left curly bracket, but ultimately I thought that an `_`(underscore) best satisfied the above defined constraints.

## File Hashes in File Names
There is an argument to be made about storing the checksum of the file in the file name itself.
ie (md5):
```
2018-01-01_SC2_tyler,john,jake_having a jolly time_d652a62f86b413c654638cbd17762888.mkv
```
Checksums offer the advantage that the integrity of the file remains accurate.
Even though I think this bit can be easily automated, I thought that it was a waste of the limited character limit that I had.
This kind of integrity check is really the role of the file system and data redundancy strategy.

## ISO8601

<img src="/images/2018/iso8601xkcd.png"/>

ISO8601 is the foundation for GRNC1.
By starting the file name with this timestamp, you are automatically chronologically sorting it.
I would argue that chronology is the best way to sort personal media files, especially when you have to sort tens of thousands of photos for example.
ISO is an international standard that can optionally support even more granular units of time (hours, minutes, seconds, and more).
Unfortunately, all the characters supported by ISO8601 are not supported by all file systems in file names.
This means that we are working with a limited version of ISO8601.
You can see more info about the standard [here](https://en.wikipedia.org/wiki/ISO_8601).

## Protocol Version in File Name
The decision to put the protocol version in the file name was considered.
Getting inspiration from a lot of networking protocols, I think that there is value in putting the protocol version in the file name.
An example might look like this (note the last field):
```
2018-01-01_HOTS_John,Jake_playing ranked games_9_f_GRNC1.mkv
```
Having the protocol version in the file name means that you could smoothly transition from an older version of the protocol to a newer one.
For example, if there was a GRNC2 that was released, but most clients still only supported GRNC1, you could easily modify old code:
```Python
if GRNC1_Detect(fileName):
	parseGRNC1(fileName)
else:
	print "I don't know how to parse that"
```
to handle the GRNC2:
```Python
if GRNC1_Detect(fileName):
	parseGRNC1(fileName)
elif GRNC2_Detect(fileName):
	parseGRNC2(fileName)
else:
	print "I don't know how to parse that"
```
Without putting the protocol version in the name of the file, you can't really do the evolution described above and you have to be more intentional about your protocol version migration strategies.

An example of networking protocols sending their version so that the request can be properly parsed can be found in HTTP requests:

 <img src="/images/2018/httpexample.png"/>

 Ultimately, I thought that this wasn't really required and it wasn't worth using the extra characters.

## Example GRNC1 Parser in Python
Here is a barebones parser I wrote in Python that you can adapt:
```Python
def parseGRNC1(fileName):
	# we are expecting that there are underscores in the name, because it is built into the protocol
	# be aware that these could be empty strings
	# we need to strip because the protocol allows for spaces at the end and beginning of spaces
	fields = String.split(fileName, "_")

	# date of the recording
	date = fields[0].strip()

	# string name of the game played
	game = fields[1].strip()

	# a list of the names of the players
	list_of_people = fields[2].strip().split(",")

	# an integer from 0 to 9 describing how much it was worth rewatching
	rewatch_factor = int(fields[0].strip())

	# a boolean, if true, the video is a full recording and not just a short clip (this is subjective)
	full_recording = fields[0].strip().lower() == 'f'
```
See? Because we were disciplined in naming the file correctly, parsing it is simple. <3

## RegEx (TODO)

## File Metadata as An Alternative
Like the database alternative, this information could also be stored in the metadata of the file.
For example, photos often have [Exif](https://en.wikipedia.org/wiki/Exif) data.
This is a fine place to store data, but I did not like how it changes when file types change.
Also it is not always easy to modify this data (especially compared to modifying a file name).
Easy of use and cross platform preservation were at stake here.

---

## More Reading:
 - [Good Style Practices for Separators in File or Directory Names](https://unix.stackexchange.com/questions/44153/good-style-practices-for-separators-in-file-or-directory-names)
 - [Plex Media Preparation](https://support.plex.tv/articles/categories/media-preparation/)
 - [Naming Files, Paths, and Namespaces on MSDN](https://msdn.microsoft.com/en-us/library/aa365247)
 - [Wikipedia: ISO8601](https://en.wikipedia.org/wiki/ISO_8601)
 - [Wikipedia: Comparison of File Systems](https://en.wikipedia.org/wiki/Comparison_of_file_systems)
 - [Wikipedia: Exif](https://en.wikipedia.org/wiki/Exif)
 - [What Characters Are Forbidden in Windows and Linux Directory Names](https://stackoverflow.com/questions/1976007/what-characters-are-forbidden-in-windows-and-linux-directory-names)
