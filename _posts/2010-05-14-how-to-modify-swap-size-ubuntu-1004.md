--- 
title: How to Modify Swap Size - Ubuntu 10.04
date: "2010-05-14T01:04:00.001-03:00"
author: William Mora
tags: 
- mkswap
- ubuntu 10.04
- linux
- lucid lynx
- swap size
- swapon
- Oracle
permalink: /2010/05/how-to-modify-swap-size-ubuntu-1004.html
---

When installing Ubuntu, I was never asked about the desired size for my swap size. Oracle recommends to use at least 2GB of memory in order to create a database, let alone run it.
With help from [this excellent post](http://www.linuxreaders.com/2009/10/28/how-to-modify-swap-size/) from [LinuxReaders](http://www.linuxreaders.com/), I expanded my swap size to 2GB (again, Oracle's recommended size for 11g) by executing the following:

```bash
$ dd if=/dev/zero of=/tmp/swapfile bs=1MB count=2048
2048+0 records in
2048+0 records out
2048000000 bytes (2.0 GB) copied, 36.257 s, 56.5 MB/s
$ mkswap /tmp/swapfile
mkswap: /tmp/swapfile: warning: don't erase bootbits sectors
       on whole disk. Use -f to force.
Setting up swapspace version 1, size = 1999996 KiB
no label, UUID=aa8617a8-9500-498e-90da-f5164190ad00
$ sudo swapon /tmp/swapfile
```

Checked my swap memory with `free -m`:

```bash
            total       used       free     shared    buffers     cached
Mem:          3895       3841         53          0       1008       1799
-/+ buffers/cache:       1033       2861
Swap:         2208          0       2208
```

And made the swap size permanent by adding the following to /etc/fstab:

```bash
/tmp/swapfile          swap            swap    defaults        0 0
```

Cheers!