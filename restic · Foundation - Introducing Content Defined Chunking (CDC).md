12 Sep 2015

This post will explain Content Defined Chunking (CDC) and how it is used by restic.

Backup programs need to deal with large volumes of changing data. Saving the whole copy of each file again to the backup location when a subsequent (usually called “incremental”) backup is created is not efficient. Over time, different strategies have emerged to handle data in such a case.

In a backup program, data de-duplication can be applied in two locations: Removing duplicate data from the same or different files within the same backup process (_inter-file de-duplication_), e.g. during the initial backup, or removing it between several backups that contain some of the same data (_inter-backup de-duplication_). While the former is desirable to have, the latter is much more important.

### Strategies

The most basic strategy is to only save files that have changed since the last backup, this is where the term “incremental” backup comes from. This way, unmodified files are not stored again on subsequent backups. But what happens if just a small portion of a large file is modified? Using this strategy, the modified file will be saved again, although most of it did not change.

A better idea is to split files into smaller fixed-size pieces (called “chunks” in the following) of e.g. 1MiB in size. When the backup program saves a file to the backup location, it is sufficient to save all chunks and the list of chunks the file consists of. These chunks can be identified for example by the SHA-256 hash of the content, so duplicate chunks can be detected and saved only once. This way, for a file containing of a large number of consecutive null bytes, only one chunk of null bytes needs to be stored.

On a subsequent backup, unmodified files are not saved again because all chunks have already been saved before. Modified files on the other hand are split into chunks again, and new chunks are saved to the backup location.

But what happens when the user adds a byte to the beginning of the file? The chunk boundaries (where a chunk ends and the next begins) would shift by one byte, changing every chunk in the file. When the backup program now splits the file into fixed-sized chunks, it would (in most cases) end up with a list of different chunks, so it needs to save every chunk as a new chunk to the backup location. This is not satisfactory for a modern backup program.

### Content Defined Chunking

Restic works a bit differently. It also operates on chunks of data from files and only upload new chunks, but uses a more sophisticated approach for splitting files into chunks called _Content Defined Chunking_. It works by splitting a file into chunks based on the contents of the file, rather than always splitting after a fixed number of bytes.

In the following, the function

$$
F(b_0…b_{63})
$$

returns a 64 bit [Rabin Fingerprint](https://en.wikipedia.org/wiki/Rabin_fingerprint) of the byte sequence in the argument (where

$$
b_i
$$

is the byte at offset

$$
i
$$

). This function can be efficiently computed as a [rolling hash](https://en.wikipedia.org/wiki/Rolling_hash), which means that

$$
F(b_1 …
b_{64})
$$

can be computed without much overhead when

$$
F(b_0 … b_{63})
$$

is already known. Restic uses 64 bytes as the “window size” for the rolling hash.

When restic saves a file, it first computes the Rabin Fingerprints for all 64 byte sequences in the file, so it starts by computing

$$
F(b_0 … b_{63})
$$

, then

$$
F(b_1 … b_{64})
$$

, then

$$
F(b_2 … b_{65})
$$

and so on. For each fingerprint, restic then tests if the lowest 21 bits are zero. If this is the case, restic found a new chunk boundary.

A chunk boundary therefore depends only on the last 64 bytes before the boundary, in other words the end of a chunk depends on the last 64 bytes of a chunk. This especially means that chunks are variable-sized, within reasonable limits.

Returning to our earlier example, if a user creates a backup of a file and then inserts bytes at the beginning of the file, restic will find the same chunk boundary for the first chunk during the second run. The content of this first chunk will have changed (due to the additional bytes), but any subsequent chunk will remain identical thanks to the content-defined chunk boundaries.

Let’s say our file consists of 4MiB of data, and restic detects the following chunk boundaries, where “offset” is the byte offset of the last byte of the sliding window:

| Offset | Fingerprint |
| --- | --- |
| 577536 | `0x77db45c60d400000` |
| 1990656 | `0xc0da6ed30fe00000` |
| 2945019 | `0x309235f507600000` |
| 4194304 | End of File |

The file is therefore split into four chunks. Adding 20 bytes at the beginning of the file still yields the same chunk boundaries, shifted by 20 bytes:

| Offset | Fingerprint |
| --- | --- |
| 577556 | `0x77db45c60d400000` |
| 1990676 | `0xc0da6ed30fe00000` |
| 2945039 | `0x309235f507600000` |
| 4194324 | End of File |

When restic computes a cryptographic hash (SHA-256) over the data in each chunk, it detects that the first chunk has been changed (we added 20 bytes, remember?), but the remaining three chunks have the same hash. Therefore, it only needs to save the changed first chunk.

### Examples

So, let’s take the things explained above to the real world, and have a bit of fun with restic. For the sake of simplicity, we’ll save the repository location and the password in environment variables (`RESTIC_REPOSITORY` and `RESTIC_PASSWORD`) so that we don’t have to type the password for every action:

```
$ export RESTIC_REPOSITORY=/tmp/restic-test-repository RESTIC_PASSWORD=foo
```

Please be aware that this way the password will be contained in your shell history.

First, we’ll initialize a new repository at a temporary location:

```
$ restic init
created restic backend 2b310bf378 at /tmp/restic-test-repository

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

At this point, nothing has been saved to the repository, so it is rather small:

```
$ du -sh $RESTIC_REPOSITORY
8.0K/tmp/restic-test-repository
```

Next, we create a new directory called `testdata` for our test, containing a file `file.raw`, filled with 100MiB of random data:

```
$ mkdir testdata
$ dd if=/dev/urandom of=testdata/file.raw bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 5.76985 s, 18.2 MB/s
```

We then backup this directory with restic (into the repository we specified via the environment variable `$RESTIC_REPOSITORY` above):

```
$ restic backup testdata
scan [/home/fd0/tmp/testdata]
scanned 1 directories, 1 files in 0:00
[0:02] 100.00%  41.457 MiB/s  100.000 MiB / 100.000 MiB  0 / 2 it...ETA 0:00
duration: 0:02, 44.21MiB/s
snapshot 7452bd17 saved
```

We can see that restic created a backup with a size of 100MiB in about two seconds. We can verify this by checking the size of the repository again:

```
$ du -sh $RESTIC_REPOSITORY
101M/tmp/restic-test-repository
```

Not surprisingly, the repository is roughly the same size as the data we have created the backup of.

Now, we run the backup command for a second time:

```
$ restic backup testdata
using parent snapshot 7452bd17
scan [/home/fd0/tmp/testdata]
scanned 1 directories, 1 files in 0:00
[0:00] 100.00%  0B/s  100.000 MiB / 100.000 MiB  0 / 2 items  ... ETA 0:00
duration: 0:00, 20478.98MiB/s
snapshot 0b870550 saved
```

Again we’ve instructed restic to backup 100MiB of data, but in this case restic was much faster and finished the job in less than a second. By the way, restic would have also been able to efficiently backup a file that was renamed or even moved to a different directory.

Looking at the repository size we can already guess that it is still about 100MiB, since we didn’t really add any new data:

```
$ du -sh $RESTIC_REPOSITORY
101M/tmp/restic-test-repository
```

When we make a copy of the file `file.raw` and backup the same repository again, restic recognises that all data is already known and the repository size does not grow at all, although the directory `testdata` now contains 200MiB of data:

```
$ cp testdata/file.raw testdata/file2.raw

$ du -sh testdata
200Mtestdata

$ restic backup testdata
using parent snapshot 0b870550
scan [/home/fd0/tmp/testdata]
scanned 1 directories, 2 files in 0:00
[0:01] 100.00%  163.617 MiB/s  200.000 MiB / 200.000 MiB  1 / 3 ... ETA 0:00
duration: 0:01, 129.06MiB/s
snapshot ab8e0047 saved

$ du -sh $RESTIC_REPOSITORY
101M/tmp/restic-test-repository
```

Now for the final demonstration we’ll create a new file `file3.raw` which is a nasty combination of the 100MiB we’ve initially saved in `file.raw` so that `testdata` now contains about 400MiB:

```
$ (echo foo; cat testdata/file.raw; echo bar; cat testdata/file.raw; echo baz) \
  > testdata/file3.raw

$ du -sh testdata
401Mtestdata
```

We’ll create a new backup of the directory with restic and observe that the repository has grown by about 10MiB:

```
$ restic backup testdata
using parent snapshot ab8e0047
scan [/home/fd0/tmp/testdata]
scanned 1 directories, 3 files in 0:00
[0:03] 100.00%  127.638 MiB/s  400.000 MiB / 400.000 MiB  2 / 4 ... ETA 0:00
duration: 0:03, 123.73MiB/s
snapshot a8897ae3 saved

$ du -sh $RESTIC_REPOSITORY
111M/tmp/restic-test-repository
```

This is expected because we’ve created a few new chunks when creating `file3.raw`, e.g. the first chunk will be saved again because a few bytes (the string `foo\n`) were added. Restic managed this challenge quite well and only introduced minor overhead for storing this incremental backup.

### Conclusion

Content Defined Chunking is a clever idea to split large amounts of data (e.g. large files) into small chunks, while being able to recognize the same chunks again when shifted or (slightly) modified.

This enables restic to de-duplicate data on the level of chunks so that each chunk of data is only stored at (and transmitted to) the backup location once. This gives us not only _inter-file_ de-duplication, but also the more relevant _inter-backup_ de-duplication.

### References

-   [The restic CDC implementation](https://github.com/restic/chunker) and [API documentation](http://godoc.org/github.com/restic/chunker)
-   Michael O. Rabin (1981): [Fingerprinting by Random Polynomials](http://www.xmailserver.org/rabin.pdf)
-   Ross N. Williams (1993): [A Painless Guide to CRC Error Detection Algorithms](http://www.zlib.net/crc_v3.txt)
-   Andrei Z. Broder (1993): [Some Applications of Rabin’s Fingerprinting Method](http://www.xmailserver.org/rabin_apps.pdf)
-   Shuhong Gao and Daniel Panario (1997): [Tests and Constructions of Irreducible Polynomials over Finite Fields](http://www.math.clemson.edu/~sgao/papers/GP97a.pdf)
-   Andrew Kadatch, Bob Jenkins (2007): [Everything we know about CRC but afraid to forget](https://github.com/rurban/crcutil/raw/master/doc/crc.pdf)