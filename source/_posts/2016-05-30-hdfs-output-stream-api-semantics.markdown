---
layout: post
title: "hdfs output stream api semantics"
date: 2016-05-30 14:00:09 -0700
comments: true
categories: [systems programming, operating systems, linux, hadoop file system, hdfs, buffer cache, page cache, close, fsync, fflush, fclose]
---

Writing to files can get tricky. You have to think about the semantics you
want, versus any performance imperatives, etc. Here, we look a briefly at the
Linux file system API, and then contrast it with a brief look at the Hadoop
File System (HDFS) Java API.

## Linux file API

In the normal Linux file system API, there are various ways to "flush" a file.
Here are a few of the ones I have seen.

We have `fflush(3)`, which flushes all user-space buffers via the
stream's underlying write function. This data may still exist in the kernel
(e.g. buffer cache and page cache [since 2.4, Linux buffer cache usually just
points to an entry in the page cache [see quora][quoraQ]]).

We have `fsync(2)`, which flushes modified pages of data from the operating
system's buffer cache to the actual disk device, and blocks until this has
completed. Modified metadata (e.g. file size) is also written out to the
file's inode's metadata section.

We have `close(2)`, which closes a file descriptor, but does not cause
flushing of any kernel buffers.

We have `fclose(3)`, which closes the file descriptor, _and_ flushes its _user
space_ buffers (like `fflush(3)`).

[quoraQ]:
https://www.quora.com/What-is-the-major-difference-between-the-buffer-cache-and-the-page-cache/answer/Robert-Love-1?srid=zA4O

### References

* [man fflush(3)](http://man7.org/linux/man-pages/man3/fflush.3.html)
* [man fsync(2)](http://man7.org/linux/man-pages/man2/fsync.2.html)
* [wiki inode](https://www.wikiwand.com/en/Inode#/POSIX_inode_description)
* [man close(2)](http://linux.die.net/man/2/close)
* [man fclose(3)](http://linux.die.net/man/3/fclose)

## Hadoop File System (HDFS) file Java API

In this API the names of the functions are similar, but the semantics are
quite different.

In HDFS, a "file" is stored as a sequence of "blocks", and each block is is
globally-configured to be e.g. either 64MB or 128MB in size. Each block is
separately stored on the configured number of machines, according to the
chosen HDFS "replication factor". For the instance of Linux running on a
particular node in the HDFS cluster, a block is a file that Linux must track
just like it would any other file: with a page/buffer cache (see above),
inode, etc. Tracking and deciding which blocks belong to each HDFS file, and
on which nodes each of those blocks are stored is the responsibility of the
HDFS NameNode (i.e. the single master node).

But the whole block-level view of HDFS is not (directly) visible to the HDFS
client API. Instead, a client simply opens an `OutputStream` to a file by
telling the name node that it either wants to create a new file, or append to
an existing file. The NameNode responds with nodes that should accept the
first block of data. The client starts writing to the first DataNode willing
to take its data. That DataNode, pipelines this incoming data to the other
DataNodes responsible for replicating this block.

Similar to the Linux file system API above, just because bytes are being
"written" by the client, does __not__ mean they'll necessarily:

1. be visible to someone who now tries to the read the file
2. be reflected in the current metadata available about the file (which lives
in the NameNode)
2. survive crashes of
1. the client or
2. the DataNode(s)

Similar to the Linux file system API above, we have a few methods we can use
to decide the buffering semantics we want of our pending writes.

We have `hflush()`, which flushes data in the client's user buffer all the way
out to the nodes which are responsible for storing the relevant _"blocks"_ of
this file. The metadata in the NameNode is __not__ updated. Data is _not_
necessily flushed from the DataNodes' buffer caches to the actual disk device.

We have `hsync()`, which is _just_ an alias for `hflush`.

We have `close()`, which closes the stream, makes sure all the data has arrived
at all the relevant nodes, and updates the metadata in the NameNode (e.g.
updates the file-length as seen from the `hadoop fs -ls myFile.txt` command
line interface).

In my experience, __it is not safe to be opening and closing the same files
from the same instance of the Hadoop client on different threads__. Maybe I
was naive in thinking this would be OK, as the Linux man pages given above
seem to suggest that this would be problematic even with the direct Linux file
system API.


### References

* [HDFS output stream API](https://hadoop.apache.org/docs/r2.6.1/api/org/apache/hadoop/fs/FSDataOutputStream.html)
