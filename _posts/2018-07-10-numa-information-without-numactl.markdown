---
layout: post
title: "How to get NUMA Information without numactl"
date: 2018-07-10
comments: false
published: True
categories: numa numactl /sys
---

I recently had a task to collect [NUMA](https://en.wikipedia.org/wiki/Non-uniform_memory_access) information from a particular system.
For a while I had been using a command called "numactl", specifically I had been calling:
```bash
numactl -H
```
and then parsing the output.
This worked great, until I was no longer allowed to use numactl!
numactl isn't necessarily going to be installed on every system, so I couldn't write plays/scripts expecting that it would be there.
I perused the [numactl man page](https://linux.die.net/man/8/numactl) to find a reference to a file in sysfs:
```bash
/sys/devices/system/node/node*/numastat for NUMA memory hit statistics.
```
 I've recently been trying to learn more about sysfs and am always surprised by the wealth of information in there.
 After a few minutes of poking around sysfs, I was able to find the information I needed.
 If you are doing write operations and want to be safe, it still might be a good idea to use the `numactl` command; but there may be times where you may not need to install numactl.
 You can see a sample of my work below (use them at your own caution ;):

 ```python
 #! /usr/bin/python

 import os
 import sys
 import re

 directory_to_numa_info = "/sys/devices/system/node/"

 def main():
 	# note that this includes both files and directories
 	try:
 		allEntries = os.listdir(directory_to_numa_info)
 	# if that directory doesn't exist or can't be listed
 	except OSError:
 		print "That directory doesn't exist! Something is wrong here..."
 		sys.exit()

 	# we ONLY want the NUMA node directories in our list
 	allEntries = [i for i in allEntries if re.match(r'node\d*', i)]

 	# we just want to double check that our target directory is indeed a directory, if it's not a directory then remove it
 	# this line could be combined with the one above, but to keep it readable, we will keep them separate
 	allEntries = [i for i in allEntries if os.path.isdir(os.path.join(directory_to_numa_info, i))]

 	numaNodeList = []

 	# now that we have our target directories, let's iterate through them to populate our numaNodeList
 	for entry in allEntries:
 		temp = NUMANode()
 		# get the number of the NUMA node
 		numaNodeNumber = re.match(r'(node)(\d*)', entry).group(2)
 		temp.nodeNumber = numaNodeNumber

 		# open the meminfo file to get some information for a given NUMA node
 		with open(os.path.join(directory_to_numa_info, os.path.join("node" + numaNodeNumber, "meminfo"))) as f:
 			# read all of the lines of our file
 			content = f.read().splitlines()
 			# check every single line
 			for l in content:
 				# this line describes the total memory for that NUMA node
 				matchGroup = re.match(r'^(Node\ \d\ MemTotal:)(\s*)(\d*)(\s*)(\w*)', l)
 				if matchGroup and matchGroup.group(5) == "kB":
 					temp.memTotalMiB = (((int(matchGroup.group(3)) * 1000) // 1024) // 1024)
 				# this line describes the total memory for that NUMA node
 				matchGroup = re.match(r'^(Node\ \d\ MemFree:)(\s*)(\d*)(\s*)(\w*)', l)
 				if matchGroup and matchGroup.group(5) == "kB":
 					temp.memFreeMiB = (((int(matchGroup.group(3)) * 1000) // 1024) // 1024)

 		# open the cpulist file to get some information about the cpus for a given NUMA node
 		with open(os.path.join(directory_to_numa_info, os.path.join("node" + numaNodeNumber, "cpulist"))) as f:
 			# read all of the lines of our file
 			content = f.read().splitlines()
 			# check every single line
 			for l in content:
 				# this is just paranoia and ensuring that the output is what I think it is
 				# I am expecting the contents of the file to looks something like this:
 				# 10-19,30-39
 				# but it could also look like this (for whatever reason)
 				# 10-19,30-39,40-100,101-20000
 				# or even a small one like this:
 				# 1-19
 				if re.match(r'^(\d*-\d*,*)*$', l):
 					sections = l.split(",")
 					for section in sections:
 						boundaries = section.split("-")
 						for core in range(int(boundaries[0]), int(boundaries[1]) + 1):
 							temp.cpuList.append(core)

 		numaNodeList.append(temp)

 	# nicely print out all of our findings
 	for numaNode in numaNodeList:
 		numaNode.prettyPrint()

 	print NUMANodesJSONString(numaNodeList)

 class NUMANode():
 	# there are many fields of a NUMA nodes that we could save, but for this example let's just take these
 	def __init__(self):
 		self.nodeNumber = -1
 		self.memTotalMiB = -1
 		self.memFreeMiB = -1
 		self.cpuList = []

 	def prettyPrint(self):
 		print "-----------------------"
 		print "NUMA Node"+str(self.nodeNumber)
 		print "Total Memory MiB:", self.memTotalMiB
 		print "Free Memory MiB: ", self.memFreeMiB
 		print "CPU List:        ", self.cpuList
 		print "-----------------------"

if __name__=='__main__':
	main()
 ```


I was able to get the information I needed without `numactl`:

```bash
tyler@myLinuxMachine:~$ ./getNUMACPUList.py
-----------------------
NUMA Node0
Total Memory MiB: 62815
Free Memory MiB:  16491
CPU List:         [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29]
-----------------------
-----------------------
NUMA Node1
Total Memory MiB: 62996
Free Memory MiB:  27
CPU List:         [10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39]
-----------------------
```


Thanks sysfs!
