---
layout: post
title: "Thundering Herd Example in Python"
date: 2018-07-02
comments: false
published: True
categories: thundering herd python threading
---

I recently wanted to stress test another program with the thundering herd problem.
You can checkout the [Wikipedia page](https://en.wikipedia.org/wiki/Thundering_herd_problem) for more info.
I just wanted to have many things to happen at the exact same time, in the hope that it could overwhelm a particular program.
Here is the code that I wrote that sets up creating a bunch of threads, all to "thunder" at the exact same time.

```python
#!/usr/bin/python

import threading
import time
import datetime
import sys

class herdMember(threading.Thread):
	def __init__(self, threadId, thunderTime):
		threading.Thread.__init__(self)
		self.threadId = threadId
		self.thunderTime = thunderTime

	def run(self):
		# wait until it's time to thunder
		while datetime.datetime.now() < self.thunderTime:
			pass
		# thunder work (just printing to STDOUT in this case)
		print "THUNDER"+str(self.threadId)

def main():
	# designate a time to thunder, 30 seconds from now
	thunderTime = datetime.datetime.now()+datetime.timedelta(seconds=30)

	# spawn our threads
	for i in range(100):
		herdMember(str(i), thunderTime).start()
		print "Started Thread:", i

	# while we have threads active (other than the main thread)
	while threading.active_count() > 1:
		time.sleep(1)
		print "Active Threads:", threading.active_count()
		# if it's not thunder time, let's count it down
		if thunderTime - datetime.datetime.now() > datetime.timedelta(seconds=1):
			print "Seconds Till Thunder:", (thunderTime - datetime.datetime.now()).seconds

if __name__=="__main__":
	main()
```


The output might look something like this:

```bash
16:58 ~/:python ThunderingHerd.py
Started Thread: 0
Started Thread: 1
Started Thread: 2
Started Thread: 3
Started Thread: 4
Started Thread: 5
Started Thread: 6
Started Thread: 7
Started Thread: 8
Started Thread: 9
Started Thread: 10
Started Thread: 11
Started Thread: 12
Started Thread: 13
Started Thread: 14
Started Thread: 15
Started Thread: 16
Started Thread: 17
Started Thread: 18
Started Thread: 19
Started Thread: 20
Started Thread: 21
Started Thread: 22
Started Thread: 23
Started Thread: 24
Started Thread: 25
Started Thread: 26
Started Thread: 27
Started Thread: 28
Started Thread: 29
Started Thread: 30
Started Thread: 31
Started Thread: 32
Started Thread: 33
Started Thread: 34
Started Thread: 35
Started Thread: 36
Started Thread: 37
Started Thread: 38
Started Thread: 39
Started Thread: 40
Started Thread: 41
Started Thread: 42
Started Thread: 43
Started Thread: 44
Started Thread: 45
Started Thread: 46
Started Thread: 47
Started Thread: 48
Started Thread: 49
Started Thread: 50
Started Thread: 51
Started Thread: 52
Started Thread: 53
Started Thread: 54
Started Thread: 55
Started Thread: 56
Started Thread: 57
Started Thread: 58
Started Thread: 59
Started Thread: 60
Started Thread: 61
Started Thread: 62
Started Thread: 63
Started Thread: 64
Started Thread: 65
Started Thread: 66
Started Thread: 67
Started Thread: 68
Started Thread: 69
Started Thread: 70
Started Thread: 71
Started Thread: 72
Started Thread: 73
Started Thread: 74
Started Thread: 75
Started Thread: 76
Started Thread: 77
Started Thread: 78
Started Thread: 79
Started Thread: 80
Started Thread: 81
Started Thread: 82
Started Thread: 83
Started Thread: 84
Started Thread: 85
Started Thread: 86
Started Thread: 87
Started Thread: 88
Started Thread: 89
Started Thread: 90
Started Thread: 91
Started Thread: 92
Started Thread: 93
Started Thread: 94
Started Thread: 95
Started Thread: 96
Started Thread: 97
Started Thread: 98
Started Thread: 99
Active Threads: 101
Active Threads: 101
THUNDER14THUNDER19THUNDER9THUNDER15THUNDER4THUNDER5
 THUNDER13
 THUNDER21
 THUNDER10

 THUNDER25
 THUNDER11THUNDER1
THUNDER28THUNDER29THUNDER31
 THUNDER32THUNDER33
 THUNDER8
THUNDER23
THUNDER2
THUNDER22
THUNDER16THUNDER18
 THUNDER0
THUNDER12

THUNDER6

 THUNDER7

THUNDER20
 THUNDER17


THUNDER3

THUNDER26
THUNDER24

THUNDER36

THUNDER27
 THUNDER35THUNDER37

THUNDER30

THUNDER34
THUNDER38
 THUNDER40THUNDER39

THUNDER41
 THUNDER45THUNDER84THUNDER88
 THUNDER42THUNDER90THUNDER51
 THUNDER52THUNDER54
 THUNDER58
 THUNDER59THUNDER61THUNDER62THUNDER92THUNDER64
 THUNDER66THUNDER68
 THUNDER70THUNDER72
 THUNDER75
 THUNDER78
 THUNDER81THUNDER83THUNDER82
 THUNDER85
THUNDER48
 THUNDER89
 THUNDER50

THUNDER55
 THUNDER56
THUNDER43THUNDER60

THUNDER65
THUNDER71
THUNDER74
 THUNDER76
THUNDER77THUNDER80

THUNDER86

THUNDER46
THUNDER47


THUNDER53THUNDER91
THUNDER63



 THUNDER73THUNDER79

 THUNDER87

THUNDER49

THUNDER44

 THUNDER67


THUNDER93
THUNDER57

THUNDER69

THUNDER94THUNDER99THUNDER95

THUNDER97
 THUNDER96
THUNDER98
```

You can see that it starts a bunch of threads and then lies in wait for thunder time.
When it's thunder time they all activate to do their work.
In this case, it was just printing to STDOUT.
But you could modify this work to be a variety of things.
