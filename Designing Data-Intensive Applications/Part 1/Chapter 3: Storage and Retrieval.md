# Storage and Retrieval

## Data Structures that Power Your Databases

logs: append only sequence of records

These are very efficient to add to but eventually take up a lot of space. They are ineffecient to 'get' from `O(n)` so indexing must be introduced.

*indexing*: keep some additional metadata on the side, which acts as a sign post and helps to locate the data you want.

Any kind of indexing will slow down 'writes' because there is nothing easier than just appending to a file but indexes will need to be updated every time you update a file.

Well chosen indexes speed up read queries but slow down writes. For this reason, databases don't usually index everything by default, but require you the application developer or database administrator ti choose indexes manually, using your knowledge of the application's typical query patterns.

### Hash Indexes

key-value stores, which are like a dictionary type, are implemented as hashmaps.

*Compaction:* If our file is append only, say the key value pairs being added are a cat video name and the number of times it has been watched `purr:1882, purr:2107`, compaction is saving only the most recent record `purr:2107`. The value gets overwritten instead of added continuously.

*Merging:* During compaction, we can combine file segments (different files) to further compact the data.

Making compaction and merging work in practice:

* *File format:* The best format for a log is binary, not CSV. First encode length of string followed by the string, that way we know the offset.
* *Deleting records:* To delete a key-value pair, append a deletion record (called a Tombstone). When log segments merge, the tombstone tells the merging process to discard any previous values for the deleted key.
* *Crash recovery:* On restart, in-memory hashmaps are lost. They can be recreated by reading the logs from beginning to end but this is time consuming. To speed up recovery store a snapshot of each segment's hashmap on disk which can be loaded into memory more quickly.
(NOTE: what does he mean by a snapshot? is this a duplicate?)
* *Partially written records:* Use *checksums* to find partially written records that result from database crashing mid append. (NOTE: What is a checksum?)
* *Concurreny control:* With files being append-only, there should be only one thread for writing, but can me multiple threads for reading. (NOTE: This creates a bottleneck if a file is being written to a lot.)

Although append-only logs seem wasteful, it is good for several reasons:

* Appending and segment merging are sequential write operations, which are generally much faster than random writes, especially on magnetic spinning-disk hard drives. To some extent sequential writes are also preferable on flash-based *solid state drives*.
* Concurrency and crash recovery are much simpler if segment files are append-only or immutable. For example, you don't have to worry about a case where a crash happened while a value was being overwritten, leaving you with a file containing part of the old and part of the new value spliced together. (NOTE: What if we had a combination of the two. Do not overwrite the most recent version but still have the ability to overwrite. This could add complexity where the most recent version is not easy to identify.)
* Merging old segments avoids the problem of data files getting fragmented over time.

Limitations:

* The hash table must fit in memory, storing it on disk has poor performance.
* Range queries are inefficient. Have to look up each key individually in the hashmaps.

SSTables and LSM-Trees do not have these limitations.

### SSTables and LSM-Trees

The above implementation had recent logs took precedence over previous logs. This implementation instead has logs sorted by key. Hence *Sorted String Tables*.