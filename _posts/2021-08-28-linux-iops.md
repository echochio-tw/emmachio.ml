---
layout: post
title: linux test HDD IOPS
date: 2021-08-28
tags: iops linux
---

有一個需求 要快一點的 HDD

以前的IT 就開了 3,000 IOPS SSD 還做 raid 0 

很快嗎?  .... 多快我沒測

我改成 RAMDISK 鐵定翻倍 ....

重點是要拿出測試工具給客人看
```
 yum install  ioping
```

開RAMDISK
```
mkdir /tmp/ramdisk
chmod 777 /tmp/ramdisk
mount -t tmpfs -o size=5G tmpfs /tmp/ramdisk
```

開始測試
```
[root@ip-172-31-35-11 ~] # cd /tmp/ramdisk
[root@ip-172-31-35-11 ramdisk] #  ioping -S64M -L -s4k -W -c 10 .
4 KiB >>> . (tmpfs tmpfs): request=1 time=3.48 us (warmup)
4 KiB >>> . (tmpfs tmpfs): request=2 time=7.61 us
4 KiB >>> . (tmpfs tmpfs): request=3 time=8.40 us
4 KiB >>> . (tmpfs tmpfs): request=4 time=4.27 us
4 KiB >>> . (tmpfs tmpfs): request=5 time=4.08 us
4 KiB >>> . (tmpfs tmpfs): request=6 time=3.82 us
4 KiB >>> . (tmpfs tmpfs): request=7 time=4.59 us
4 KiB >>> . (tmpfs tmpfs): request=8 time=3.84 us (fast)
4 KiB >>> . (tmpfs tmpfs): request=9 time=3.68 us (fast)
4 KiB >>> . (tmpfs tmpfs): request=10 time=4.39 us

--- . (tmpfs tmpfs) ioping statistics ---
9 requests completed in 44.7 us, 36 KiB written, 201.4 k iops, 786.7 MiB/s
generated 10 requests in 9.00 s, 40 KiB, 1 iops, 4.44 KiB/s
min/avg/max/mdev = 3.68 us / 4.96 us / 8.40 us / 1.66 us
[root@ip-172-31-35-11 ramdisk] # cd /root
[root@ip-172-31-35-11 ~] #  ioping -S64M -L -s4k -W -c 10 .
4 KiB >>> . (xfs /dev/nvme0n1p1): request=1 time=778.3 us (warmup)
4 KiB >>> . (xfs /dev/nvme0n1p1): request=2 time=771.3 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=3 time=583.9 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=4 time=671.0 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=5 time=888.0 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=6 time=755.6 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=7 time=723.8 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=8 time=1.05 ms (slow)
4 KiB >>> . (xfs /dev/nvme0n1p1): request=9 time=659.2 us
4 KiB >>> . (xfs /dev/nvme0n1p1): request=10 time=638.2 us

--- . (xfs /dev/nvme0n1p1) ioping statistics ---
9 requests completed in 6.74 ms, 36 KiB written, 1.33 k iops, 5.21 MiB/s
generated 10 requests in 9.00 s, 40 KiB, 1 iops, 4.44 KiB/s
min/avg/max/mdev = 583.9 us / 749.0 us / 1.05 ms / 135.4 us
```

大檔案超快的
```
[root@ip-172-31-35-11 ramdisk]# ioping -S1G -L -s10M -W -c 10 .
10 MiB >>> . (tmpfs tmpfs): request=1 time=2.27 ms (warmup)
10 MiB >>> . (tmpfs tmpfs): request=2 time=2.69 ms
10 MiB >>> . (tmpfs tmpfs): request=3 time=2.66 ms
10 MiB >>> . (tmpfs tmpfs): request=4 time=2.64 ms
10 MiB >>> . (tmpfs tmpfs): request=5 time=2.66 ms
10 MiB >>> . (tmpfs tmpfs): request=6 time=2.63 ms
10 MiB >>> . (tmpfs tmpfs): request=7 time=2.69 ms
10 MiB >>> . (tmpfs tmpfs): request=8 time=2.64 ms
10 MiB >>> . (tmpfs tmpfs): request=9 time=2.62 ms (fast)
10 MiB >>> . (tmpfs tmpfs): request=10 time=2.63 ms

--- . (tmpfs tmpfs) ioping statistics ---
9 requests completed in 23.8 ms, 90 MiB written, 377 iops, 3.69 GiB/s
generated 10 requests in 9.01 s, 100 MiB, 1 iops, 11.1 MiB/s
min/avg/max/mdev = 2.62 ms / 2.65 ms / 2.69 ms / 24.4 us
[root@ip-172-31-35-11 ramdisk]# cd
[root@ip-172-31-35-11 ~]# ioping -S1G -L -s10M -W -c 10 .
10 MiB >>> . (xfs /dev/nvme0n1p1): request=1 time=19.5 ms (warmup)
10 MiB >>> . (xfs /dev/nvme0n1p1): request=2 time=18.9 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=3 time=19.4 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=4 time=19.4 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=5 time=18.8 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=6 time=18.8 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=7 time=20.2 ms (slow)
10 MiB >>> . (xfs /dev/nvme0n1p1): request=8 time=19.2 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=9 time=19.4 ms
10 MiB >>> . (xfs /dev/nvme0n1p1): request=10 time=19.6 ms

--- . (xfs /dev/nvme0n1p1) ioping statistics ---
9 requests completed in 173.8 ms, 90 MiB written, 51 iops, 517.9 MiB/s
generated 10 requests in 9.02 s, 100 MiB, 1 iops, 11.1 MiB/s
min/avg/max/mdev = 18.8 ms / 19.3 ms / 20.2 ms / 430.3 us
```
