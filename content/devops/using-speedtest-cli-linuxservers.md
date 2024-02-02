---
title: "Checking your internet connection with speedtest-cli "
date: 2012-08-04T14:16:40+01:00
tags: ["Linux"]
summary: "Using speedtest-cli on linux debian headless"
draft: false
---


### Install speedtest-cli

{{<highlight bash>}}

sudo apt-get install python-pip
sudo pip install speedtest-cli

{{</highlight>}}



### And now we test

{{<highlight bash>}}

speedtest-cli

{{</highlight >}}


If you wish to share you results just run with:


{{<highlight bash>}}

speedtest-cli --share

{{</highlight >}}
