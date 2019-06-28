---
title: NeverLAN - JSON parsing 1
layout: post
date: '2018-02-27 12:34:00'
description: Writeup of JSON parsing 1
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Kauê Doretto
---

### JSON parsing 1 (200)

Description:
> The attached file is metadata about one minute's uploads to VirusTotal.
> 
> The answer to this puzzle is a comma-separated list of the five antivirus engines that produced the highest percentage of posities in descending order.
> 
> Don't draw any conclusions about the efficacy of any antivirus products from this exercise- VirusTotal receives a mixture of malicious and non-malicious files, so it's not necessarily better to have a high ratio than a low one or the other way around here. I also have no associated with any of them. It's just a data manipulation puzzle, people =)
> 
> NOTE: The answer should be submitted with no spaces and the engine names should be exactly as they appear in the source data.
> 
> [vt_minute_output.tar.bz2](https://s3-us-west-2.amazonaws.com/neverlanctf/files/vt_minute_output.tar.bz2){:target="_blank" rel="noopenner noreferrer"}


---
1.Download the file from the URL and extract it 
```
$ bzip2 -d vt_minute_output.tar.bz2
$ tar -xvf vt_minute_output.tar
./._file-20171020T1500
file-20171020T1500
```

2.The file is in the json format, so we'll have to write a python script to extract, classify and print the data. Note that the challenge is not asking the "list of the five antivirus engines that produced the highest positives count", it's asking for the ratio of each individual engine, calculated from the total number of positives divided by the total number of files processed by it. So and engine that has a ratio of 1/5 will be positioned higher in the list than one with 130/800.
 
3.The program will iterate over all json objects in the file, creating a python dictionary variable with all antivirus engines found. Each engine will have a "total" and a "detected" variables associated. At the end of the json file's processing, we will print them in ascending order of total/detected ratio, so that the engines we are looking for will be closer to the prompt when it's outputted.

4.The json object's part with the data we're looking for is this one. So we're extractig the value of the "detected" key
```
"scans": {
    "Bkav": {"detected": false, "version": "1.3.0.9367", "result": null, "update": "20171020"}, 
    "K7AntiVirus": {"detected": false, "version": "10.29.24984", "result": null, "update": "20171019"} 
    [...]
```

5.This is the python program source
```python
#!/usr/bin/python3.6

import json
import operator
import pprint

av_list = []
fp = open("file-20171020T1500")

# Iterate over the objects and store them in a list
for line in fp.readlines():
    av_list.append(json.loads(line))

av_dict = {}

for submit in range(0, len(av_list)):

	for antivirus in av_list[submit]["scans"].keys():

		# First time the engine is found, create the dict item using
		# the av engine as the key
		if antivirus not in av_dict:
			av_dict[antivirus] = {}
			av_dict[antivirus]["total"] = 0
			av_dict[antivirus]["detected"] = 0

		# Increment the total of scans
		av_dict[antivirus]["total"] += 1

		# Increment the detected total
		if av_list[submit]["scans"][antivirus]['detected']:
			av_dict[antivirus]["detected"] += 1

# Sort the dict by the detected/total ratio
sorted_av_dict = sorted(av_dict.items(), 
    key=lambda x: 100 * float(int(x[1]["detected"])) / float(int(x[1]["total"])))

# In case you want to see the list
# pprint.pprint(sorted_av_dict)

# The answer will be the last 5
start = len(sorted_av_dict) - 1
stop = len(sorted_av_dict) - 6

# Comma separated values, no spaces
# Remove manually the last comma
for x in range(start, stop, -1):
	print(sorted_av_dict[x][0] + ",", end='')

print("")
```

6.The output (remove the last comma)
```
$ ./js200.py 
SymantecMobileInsight,CrowdStrike,SentinelOne,Invincea,Endgame,
```
