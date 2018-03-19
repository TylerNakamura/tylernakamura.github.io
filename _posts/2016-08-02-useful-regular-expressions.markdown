---
layout: post
title: "Useful Regular Expressions"
date: 2016-08-02 13:49
comments: false
categories: Regex regular expression
---

Just saving some RegEx that I use :D

### Season and Episode:
```bash
[Ss]\d{2}[Ee]\d{2}
```
Example:

- S01E10
- s11E12
- S30e01

### Video Types
```bash
*\.(webm|mkv|flv|vob|ogv|ogg|drc|gif|gifv|mng|avi|mov|qt|wmv|yuv|rm|rmvb|asf|amv|mp4|m4p|m4v|mpg|mp2|mpeg|mpe|mpv|mpg|mpeg|m2v|m4v|svi|3gp|3g2|mxf|roq|nsv|f4v|f4p|f4a|f4b)
```

Note that in the above example, it is case sensitive.
I opted to not imlement a case sensitive RegEx as it varies between RegEx implementations.
See [here](https://stackoverflow.com/questions/9655164/regex-ignore-case-sensitivity) for more details.
See [this page](https://en.wikipedia.org/wiki/Video_file_format) on Wikipedia for an explanation on extension types.

Example matches:

- mymovie.mkv
- sickclip.mp4
- stuff.avi


### ipv4 address
```bash
^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}↵
(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$
```
[source](https://www.safaribooksonline.com/library/view/regular-expressions-cookbook/9780596802837/ch07s16.html)

### URLS
```bash
https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)
```
[source](http://stackoverflow.com/questions/3809401/what-is-a-good-regular-expression-to-match-a-url)
