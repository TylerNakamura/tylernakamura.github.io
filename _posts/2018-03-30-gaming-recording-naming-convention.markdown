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
|Game|Name of Game Played | `Runscape` `Fornite` `FN` `SC2` `WoW` |
||||
|Players| Comma separated list of player names| `tyler` `todd, billy joel, sam` |
||||
|Description| Description of the event| `fun` `awesome fun times` `that one time that John farted` `ranked games`|
||||
|Rewatch Value|  A number from 0 to 9 describing your desire to rewatch later. 9 is most rewatch worthy | `0` `5` `9`|
||||
|Full or Partial Recording| Distinguish between highlights and VODS, 0 is a VOD, 1 is a highight| `0` `1`  |
||||

### Rules:
- Use a single `_`(underscore) to delimit each field.
- Empty fields are acceptable, but still use the delimiter.
- Spaces are tolerated, they are _not_ being used to delimit anything, so go crazy with them.
- While ISO8601 is required, not all characters in the standard are allowed by all file systems. So we have to work with a limited version of ISO8601. Check your file system constraints.

Valid Examples:
```
2018-01-01 _ HOTS _ Billy,John Michael,Jake, Sally _ Winning a ton of ranked games_GRNC1.mp4
2018-01-xx _ FN _ Tyler, Bean _GRNC1.mp4
2018 _ FN__GRNC1.mkv
2018_                _ _                     _               _.mkv
2018_____.mkv
_____.mkv
```

---

## Motivation

Over the last 3 years, I have been recording all of my gaming sessions with friends.
I'm not sure why I started this, but I feel that one day these old videos may be of use.
It's possible that when my brain is too slow, and my fingers too feeble, that I may want to go back to the glory days and rewatch hours of gaming footage.
I don't know, but its a hobby of mine to collect these videos and categorize them.

Now that I have accumulated hundreds of videos and TBs worth of footage, with many different games, and many different people, the demand for a proper naming convention is greater than ever.
By saving things like, the game that was played, and who it was played with allows me to easily search through my archives.
Gaming footage of my session with Bob will not be relevant or interesting to Alice, and vice versa.

I have plans on creating some kind of interface or search engine for this archive.
The first step in doing this, is properly naming these files.
That way, if I want to search for videos in 2016, played with Bob, playing World of Warcraft; I can do that.

## Using a Database Instead of Overloading File Names

I think an argument could be made that as long as each file name is unique, I could use a proper database to map these files to a row.
This would allow for proper data storage, a significantly larger amount of information stored for each video.
While I considered this, I think that there wasn't enough information that I could store for each video for a database to be worth it.
I chose instead to go with the simpler approach of storing all of this information in the file name itself.
This way, I don't have an additional data set to worry about.


## Character Limits in File Names across File Systems
Cross platform viability is important to me.
Operating systems evolve, along with their file systems.
It is important to me, that if I wanted to move the storage of these videos onto a different filesystem, I would not be hindered by the size of the file name.
Luckily, it seems like these limits are largely the same for the file systems I considered.

Here are the file systems that I wanted to account for, you can see that luckily they are pretty similar:

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

In case you are curious for more, check out this [Wikipedia page on a comparison of file systems](https://en.wikipedia.org/wiki/Comparison_of_file_systems).

## Choosing a Field Delimiter
Choosing the delimiter was the most challenging aspect of coming up with a naming convention.
While many delimiters work fine, I was especially picky about achieving the ideal one.
An obvious candidate is the `-`.
This is similar to what Plex uses in their chosen Television naming convention, it looks like this:
```
The Big Bang Theory - s04e10 - The Hot Troll Deviation.mkv
Friends - s01e01.mkv
```

While this delimiter is great for these kinds of files, it doesn't make sense to do this for gaming recordings.
This is mainly because the we require the ISO8601 dating standard in GRNC1.
ISO8601 makes use of `-`s as its own delimiter.
Plex doesn't incorporate that, so we have to find our own delimiter that transcends this.
For example, this file name would completely throw off most parsers:
```
2018-01-01 - FN -Tyler, Bean -.mp4
```
It would be difficult and overly cumbersome to try and distinguish ISO8601 dashes and GRNC1 dashes.
A `-` will not work well for GRNC1.

This means that we have to find a delimiter that satisfies these constraints:
1. Won't ever be a desired character in any of the fields as it will mess up parsing these file names
2. Is an acceptable character in _commonly used_ filesystems (FAT, NTFS, EXT3/4, ZFS)
3. Visually appealing
4. Easy to produce
5. Plays nicely with Bash

There is a lot to think about here, and I recommend reading [this post](https://stackoverflow.com/questions/1976007/what-characters-are-forbidden-in-windows-and-linux-directory-names) if you are interested in learning more.
But to keep it simple, let's just say we can't use:
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
I strongly considered a right/left curly bracket, but ultimately I thought that an underscore best satisfied the above defined constraints.

## File Hashes in File Names
There is an argument to be made about storing the checksum of the file in the file name itself.
ie (md5):
```
2018-01-01_SC2_tyler,john,jake_having a jolly time_d652a62f86b413c654638cbd17762888_GRNC1.mkv
```
Checksums offer the advantage that the integrity of the file remains accurate.
Even though I think this bit can be easily automated, I thought that it was a waste of the limited character limit that I had.
This kind of integrity check is really the role of the file system and data redundancy strategy.

## Distinction Between Highlights and Full Recordings
Note the distinction between a "highlight" and a VOD. It should be clear that this protocol is for VODs and NOT highlights.

## Automation
While I appreciate automation, I don't think it's feasible at this point to _fully_ automate naming to GRNC1.
GRNC1 seeks information about what game was played, who the game was played with, and possible enjoyment levels; all of which would require complex scraping to acquire.
You can still derive a valid GRNC1 name by just saving the ISO8601 timestamp:
```bash
2018-01-01_____.mp4
```

## ISO8601

<img src="/images/2018/iso8601xkcd.png"/>


ISO8601 is the foundation for GRNC1.
By starting the file name with this timestamp, you are automatically chronologically sorting it.
I would argue that chronology is the best way to sort personal media files, especially when you have to sort tens of thousands of photos for example.
ISO is an international standard that can optionally support even more granular units of time (hours, minutes, seconds, and more).
For the love of God, if there is one thing you should take away from this entire document, it's that you should name your files starting with a proper ISO8601 stamp.

## Protocol Version in File Name
The decision to put the protocol version in the file name was considered.
Getting inspiration from a lot of networking protocols, I think that there is value in putting the protocol version in the file name.
The main reason is that you would have a disjoint between parsers, in that older parsers of the string haven't been updated.
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

## Example GRNC1 Parser in Python
Here is a barebones parser I wrote in Python that you can adapt:
```Python
def parseGRNC1(fileName):
	# we are expecting that there are underscores in the name, because it is built into the protocol
	fields = String.split(fileName, "_")
	# we need to strip because the protocol allows for spaces at the end and beginning of spaces
	date = fields[0].strip()

```
See? Because we were disciplined in naming the file correctly, parsing it is simple. <3

## Example GRNC1 RegEx

## File Metadata?
- Ease of entry
- cross platform

---

## More Reading:
 - [Good Style Practices for Separators in File or Directory Names](https://unix.stackexchange.com/questions/44153/good-style-practices-for-separators-in-file-or-directory-names)
 - [Plex Media Preparation](https://support.plex.tv/articles/categories/media-preparation/)
 - [Wikipedia: ISO8601](https://en.wikipedia.org/wiki/ISO_8601)
 - [What Characters Are Forbidden in Windows and Linux Directory Names](https://stackoverflow.com/questions/1976007/what-characters-are-forbidden-in-windows-and-linux-directory-names)
 - [Naming Files, Paths, and Namespaces on MSDN](https://msdn.microsoft.com/en-us/library/aa365247)
 - [Wikipedia: Comparison of File Systems](https://en.wikipedia.org/wiki/Comparison_of_file_systems)
