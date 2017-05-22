# Performance basics lab

# Goals

This is a basics lab, meaning for some of you, it will probably be pretty rudimentary.  That said, here are things you should be able to take away at the conclusion of this excercise:

* Unix tools to obtain configuration and environmental details about a host
* Tools to test connectivity to other hosts
* Tools and methods to test filesystem performance
* Basic techniques to flush caches and script simple tests
* Observe behavior of applying different parameters (block size & fileSize) during a test 


# Environment


# Lab instructions



## Lay of the land


### Host environmentals 
Whenever you are either troubleshooting an issue, or are wanting to benchmark performance, its always a good idea to first make sure you understand as many details as possible about the host you are on.  This section of the lab focuses on basics, such as networking and mounted filesystems.

#### Determine network details of your host


1. Get info about all network interfaces

    ifconfig -a

You've probably used this tool many times, but how often do you pay attention to every field?  You can easily use it to get things like:
* IP address(es), subnet mask
* MTU size (this will be important in future labs)
* any RX/TX errors
* Total bytes & packets sent through this interface

    ifconfig -a | grep inet

3.  connection speed of ifaces

    ethtool devid
Note that ethtool usage can vary from system to system, you'll want to experiment on other systems to make sure you know what to expect.  This tool is useful to make sure you know:
* Link detected?
* fiber/copper 
* Negotiated speed 

More advanced uses include setting interface specific parameters such as flow control, tcp-segment-offload, and more.  Be careful when using flags which can change settings, as they may not always do what you expect..



4.  What fileSystems are mounted
 In the excercise today, we'll be focusing on locally and NFS mounted filesystems.
    ifconfig -a | grep inet

    mount
    df


### Fileserver environmental discovery (~5 mins)

1.  route to fileserver

     ip route get $NFS_SERVER_IP

2.  latency to fileserver
    
    ping $NFS_SERVER_IP


    
2.  What mounts are available on the FS? 

    showmount -e $NFS_SERVER_IP 

3.  Mount the NFS server

    mkdir /mnt/nfs
    mount -t nfs -o tcp,rw,vers=3 $NFS_SERVER_IP:/export /mnt/nfs 

We'll save more indepth discussion of NFS mount opts for another session, as there are cases where you will want to experiment with things like rsize/wsize, readahead, and others.  For now, we want to make sure we're using TCP as the transport (as oppoed to UDP), and NFSv3 (as opposed to the default in RHEL-7, which is now NFSv4). 

4.  Verify mount

    df -h 
    mount | grep nfs

Note that the `mount` command may show you some additional options.  These represent defaults for this system.  Any time you are running tests or cataloging an environment, its important to make note of mount options for accurate and valid reproduction of results and issues.

#### Basic File System tests (~10-15 mins)
Now that you have an NFS server mounted, you want to ensure that you can Create/read/write and delete files. 

1.  Make a 0byte file, just to validate you can:
        touch /mnt/nfs/testfile
Its useful to follow this with `ls -l /mnt/nfs/testfile` which will tell you:
* the user/group that you are being mapped to (helpful if you are troubleshooting root-squash issues)
* The effective umask


2.  Write a 100MB file
We'll use the `dd` command here, which is a simple way to create files of varying length quickly.  Note that using `/dev/zero` as the source is the fastest peforming, however when running tests against Fileservers/Filesystems which have inline dedupe or compression: the results can be misleading.  In those cases, you'll want to use `/dev/random` or `/dev/urandom` (faster), as `/dev/zero` is basically writing binary 0's.

        dd if=/dev/zero of=/mnt/fileserver/100MB.file bs=1M count=100
    * How fast did it go?
    * What if you re-write the same file, faster or slower?

3.  Read back the 100MB file
        dd if=/mnt/fileserver/100MB.file bs=1M
    * How fast did it go?
    * does that seem realistic..?

4.  Flush client side cache

When dealing with both local and NFS mounted filesystems, the linux page cache is utilized, and can cause misleading results.  For instance, when performing a WRITE to an NFS server, this cache is populated, such that subsequent reads will be read from cache, which lives in local RAM.  Therefore, if you are trying to get an accurate reading of performance, its critical to flush this client-side cache between operations.

There's are (at least) two ways to do this:
    * Without remounting:
            sudo echo 3 > /proc/sys/vm/drop_caches
    * By unmounting & remounting:
        sudo umount -lf /mnt/fileserver && mount -a
Note that unmounting/remounting can cause other operations from this client to have issues against the NFS server, so its typically only recommended to do this on hosts that are exclusively for your own use.  Using the former option of dropping the cache is preferred, but also can have impact to application performance because it is a system wide operation, effectively flushing the entire linux page cache.

5.  Re-run your read test
        dd if=/mnt/fileserver/100MB.file bs=1M
    * Speed change?
    *** keep in mind: even though the client cache is flushed, the NFS server cache may be giving you results which aren't realistic.***

6.  Try a bigger file:
        dd if=/dev/zero of=/mnt/fileserver/4GB.file bs=1M count=4096

Then flush, read, record results.




Measure throughput and/or latency
Advanced testing: (~10 mins)
Vary block size/threadcount/filesize
metadata operations

#### Advanced File System tests (~10-15 mins)

* Vary block size in DD

So far you've just done testing with relatively large block sizes (1MB).  However, in the real world things don't typically line up this way.  For instance, many databases (such as postgres/mysql) will use much smaller units, such as 4K or 8K.  You can experiment and see how different block sizes behave. 

eg:

    dd if=/dev/zero of=/mnt/nfs/mysql.dat bs=4k count=$((1024*1024))
    This will make a 4GB file, using 4Kilobyte block sizes.

Note that in some cases, you may not see a large difference in performance when changing the block size.  There are a number of reasons for this, one being that for sequential operations, NFS clients will batch up requests instead of sending each 4KB write request across the wire separately.  That's where the 'wsize' option comes in.

* The 'direct' flag
What if you want to bypass the client cache for reads & writes?  Also..what if you want the client to send a separate request for each 4K write request?  On linux, you have the 'direct' flag, which does a couple things:

1. Bypasses the client side writeback cache
2. Alerts the NFS server that the client wishes every WRITE call to be committed to stable storage before the server responds saying "all clear".

    dd if=/dev/zero of=/mnt/nfs/mysql.dat bs=4k count=$((1024*1024)) oflag=direct



* concurrency?
* logging?
*    

## appendix

For future labs:
* ping w/ don't fragment set (mtu)
* nmap to determine open ports
* 
