# Building a Database From Scratch Notes

## Updating Files

1. Simple writes
   - Will update the whole content
   - Read and modify inline, app crashes?
   - Concurrent access to data
2. Atomic renaming
   - Writes a new file
   - Old and new file, can recover from old
   - Non half written data on concurrency
   - "X is atomic with respect to what?"
     - Rename is atomic w.r.t concurrent readers
     - Rename is NOT atomic w.r.t power loss

### Why does renaming work?

- FS keep a mapping from file names to file data. Replacing a file by renaming \
  simply points the file name to the new data without touching the old data.
- Linux Systems
  - replaced old files may still exist if it's still being opened by a reader
    - will not be accessed from a file name
  - readers can work on opened versions
  - writers will not be blocked by readers
  - level of concurrency

### Append-only logs

- Incremental updates can be done by appending updates to a file
- The file is treated as a "log" for its append-only feature
- Safer tha in-place updates, no data is overwritten and can recover from crashes

### Atomic log updates with checksums

A log won't corrupt old data, if data is corrupted on crash it has to be validated.

- Last append does not happen, the log still good.
- Last entry is half written
- Size of the log is increased but last entry does not exist

A way is to add checksums to each log and verify the checksums.

## Indexing Data Structures

### Types of queries

#### 3 Types

1. Scan the whole data set. (No index is used)
2. Point query: Query the index by a specific key.
3. Range query: Query the index by a range. (The index is sorted).

Column-based storage can make scanning fast. Scans will always be O(N)

Queries that can be O(log N), range query consists of 2 phases:

1. Seek: find the starting key.
2. Iterate: find the previous/next key key in sorted order.

Point query is just seek without iterate

### Hashtables

Hashtables are only considered for point queries (get, set, del).

### Sorted arrays

- Binary search in O(log N)
- Updates are O(N)
- Updates can be reduced by dividing the array into non overlapping arrays (B+tree)
- Updates can be reduced with a log-structured merge tree

### B-tree

Disk-based storage systems primary bottleneck is Input/Output operations per seconds.

Each level traversal in a tree typically requires a new disk read operation.\
Tree nodes are stored in different locations on disk.
The systems steps are:

1. Read a node
2. Determine which child to access
3. Seek to that location
4. Read again

#### Binary vs. B-Trees

- A binary tree (n=2) with 1 million keys has a height of approximately\
  log_2(1,000,000) = 20
- A B-Tree with n=100 would have a height of only log_100(1,000,000) = 3 levels

B-Tree lookup might require only 3 disk reads instead of 20.

##### Tradeoffs

- Larger tree nodes are slower to update.
- Read latency increases for larger nodes.

#### IO in the unit of pages
