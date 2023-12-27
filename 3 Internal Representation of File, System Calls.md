### Inode
* Short for Index Node
* It is found in disk
* Contains Details of File in disk such as:
    * Owner
    * Group
    * Permission
    * Last Accessed - File
    * Last Modified - File
    * Last Inode Acessed
    * Size
    * File Layout in disk

* It is loaded into memory, to be used by processes.
* In in-core copy contains the above, along with
    * Status
    * Files System number
    * Inode number?
    * References count
    * Pointers to doubly linked freelist
    * pointers to doubly linked hash queue.

we have the following algorithms:
* iget
* iput
* bmap
* namei
* ifree
* ialloc
* free
* alloc 

### 1) Getting Inode
**iget(filesystem, inode number): inode**

    while inode not found:

        if inode in inode-cache:
            if inode busy:
                sleep and continue
            else:
                if inode in freelist:
                    remove from freelist
                return inode

        else:
            if no free inode in free-list:
                return error

            get a inode from freelist
            change inode number and filesystem of inode
            put inode into new queue
            read from disk using bread
            reference count set to 1
            return inode

### 2) Releasing Inode
**iput(inode):None**

    lock inode
    decrement reference count
    if reference count 0:
        if hard links 0:
            free data
            status to 0
            ifree inode
        if recently used:
            update to disk
    unlock inode

### Structure of regular file:
* Sequential blocks can fragment disk
* so non sequential way by using inode
* Contains:
    * 10 direct block
    * 1 1-d block
    * 1 2-d block
    * 1 3-d block 
* here n-d refers to the n indirection
* The size of disk block is chosen to:
    * maximize data read in one read
    * avoid fragmentation
* Also if file very small, data block could be added to inode itself.
* The following layout for block size nK bytes and m number of direct blocks per level of indirection:

**NOTE:** for a file system with logical block of size 1K Byte and addressing of 32 bit(4 Byte), we can access 256 blocks using a single indirection.

### Structure of a directory:
The directory file has the inode number and the component name.

|Part|Size|
|-|-|
Inode number| 2 byte (16 bits = 2^16 numbers)
File name|14 byte

* 16 bytes total for each directory entry.

### 3) byte offset map
**bmap(inode,byte offset): logical block address, start address, bytes to read, read ahead block**

    calculate logial block address
    calculate start address
    calculate bytes to read
    Find if read ahead possible
    calculate indirection
    while more indirection:
        go into inode or indirect block
        find the disk block address
        brelse for temporary indirect block
        if no more indirection
            return disk block
        bread indirect block into memory
        change logical address in inode

### 4) Namei
**namei( pathname): inode**

    if pathname starts with root:
        working inode is iget(root)
    else:
        working inode is iget(current directory)
    
    while more path:
        get the next component
        check access permission of inode
        read all of the inode using bmap, bread, brelse
        if component found:
            working inode = iget(current component)
        else:
            return no inode
    return working inode

### Super block

* The filesystem contain the super block
* super block has :
    * filesystem size
    * number of :
        * free inodes
        * free disk blocks
    * pointers to :
        * free inode list
        * free disk block list
    * status of all inodes and disk blocks
    * Superblock modified flag
    * Superblock lock flag

### 5) Ialloc
* Get me a free inode from superblock

**Ialloc(filesystem): locked inode**

    while inode not returned
        if super block busy
            sleep and continue
        if no free list of inode in superblock:
            lock superblock
            search from rover
            add free inodes from inode array until free list full or no more free inode(bread and brelse?)
            unlock superblock
            if no inode in free list:
                return nothing
            set rover
        if inode in free list found:
            get inode number
            iget inode
            if inode found to be not free?:
                write back to disk
                release inode?
                continue
            initialize inode
            decrement free list count
            return inode 

### 6) Ifree
**ifree(filesystem, inode number)**
    
    increment free inode count
    if superblock locked:
        return
    if inode full:
        if inode number less than rover:
            rover = inode
    else:
        store in inode list

### Last Disk Blocks:
* If free disk block list contain one block, then it is assumed to contain a list of blocks within itself

### 7) alloc

    while super block locked
        sleep
    remove block from free list
    if block LAST one:
        lock super block
        read the block (bread)
        add all the block number to free list
        brelse for block
        unlock super block
    getblk for block
    clear buffer
    decrement free block count
    super block modified flag set
    return block
        
### 8) free


NOTE : ialloc, ifree and alloc have some ambiguity in them

## System calls for the Files:

1) open
    fd = open(pathname, flag, mode)

        inode = namei(pathname)
        if no permission:
            return error
        create file table for inode, reference count, offset
        create descriptor entry to this file table entry
        if truncate file mode:
            remove all disk blocks
        unlock inode
        return descriptor
2) read

3) create
    fd = creat(pathname, mode)

        inode = namei(pathname)
        if inode exist:
            if no permission:
                error
            else: 
                remove all blocks in inode
        else:
            ialloc inode
            add directory entry in parent directory disk block
        create file table entry for inode
        create descriptor fd for inode
        unlock inode
        return fd
4) special create

5) link

6) unlink