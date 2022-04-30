---
layout: post
title: Check RTMP is live or not
date: 2021-11-11
tags: rtmp
---

Check RTMP is live or not

```
#yum install openssl-devel
#yum install zlib
#wget http://rtmpdump.mplayerhq.hu/download/rtmpdump-2.3.tgz
#tar xvf rtmpdump-2.3.tgz
#cd rtmpdump-2.3
#make
#make install
#rtmpdump -v -r rtmp://giraldatvlivefs.fplive.net/giraldatvlive-live/stream001 -o /tmp/rtmp-checker.log
RTMPDump v2.3
(c) 2010 Andrej Stepanchuk, Howard Chu, The Flvstreamer Team; license: GPL
Connecting ...
INFO: Connected...
Starting Live Stream
INFO: Metadata:
INFO:   duration              0.00
INFO:   width                 720.00
INFO:   height                1288.00
INFO:   videodatarate         878.91
INFO:   framerate             15.00
INFO:   videocodecid          7.00
INFO:   audiodatarate         46.88
INFO:   audiosamplerate       44100.00
INFO:   audiosamplesize       16.00
INFO:   stereo                TRUE
INFO:   audiocodecid          10.00
INFO:   streamId              A4D55136F9A44A25E2967BDF4DB162A0
INFO:   manufacturer          KSY-a-v5.0.1.3
INFO:   interval              9999
INFO:   utcstarttime          1635578097728
INFO:   encoder               Lavf57.71.100
INFO:   filesize              0.00
307.333 kB / 10.21 sec

```
