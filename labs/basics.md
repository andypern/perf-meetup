# Performance basics lab

# Goals


# Environment


# Lab instructions



## Lay of the land


### Host environmentals 

#### Determine network details of your host
1. all network interfaces
    ifconfig -a
2. all ip addresses
    ifconfig -a | grep inet

3.  connection speed of ifaces
    ethtool devid

4.  What fileSystems are mounted
    mount


### Fileserver environmental discovery (~5 mins)
* route to fileserver
* what protocol is being used to mount
* latency to fileserver
* what ports are open between host=>fileserver? 

#### Basic File System tests (~10-15 mins)
Create/read/write files

1.  Make a 0byte file, just to validate you can:
    touch /mnt/fileserver/testfile

2.  Write a 100MB file
    dd if=/dev/zero of=/mnt/fileserver/100MB.file bs=1M count=100
    * How fast did it go?
    * What if you re-write the same file, faster or slower?

3.  Read back the 100MB file
    dd if=/mnt/fileserver/100MB.file bs=1M
    * How fast did it go?
    * does that seem realistic..?

4.  Flush client side cache
There's two ways to do this:
    * Without remounting:
        sudo echo 3 > /proc/sys/vm/drop_caches
    * Remount:
        sudo umount -lf /mnt/fileserver && mount -a
5.  Re-run your read test
    dd if=/mnt/fileserver/100MB.file bs=1M
    * Speed change?
    ** keep in mind: even though the client cache is flushed, the NFS server cache may be giving you results which aren't realistic.

6.  Try a bigger file:
    dd if=/dev/zero of=/mnt/fileserver/4GB.file bs=1M count=4096



Measure throughput and/or latency
Advanced testing: (~10 mins)
Vary block size/threadcount/filesize
metadata operations

#### Advanced File System tests (~10-15 mins)

* Vary block size in DD
* 
