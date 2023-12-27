### The Buffer Cache:
* Used to make Read and write to files more efficient.
* Has 2 component:
    1) Buffer Header
    2) Buffer Array
* Buffer Header contains
    * Mapping to Buffer Array
    * File System Number - *logical*
    * Block Number - *logical*
    * Pointers for doubly-linked hash queue.
    * Pointers for doubly-linked free list.
* Buffer array is part of memory that contain data.

* Only one buffer per block of a Filesystem.

* Five Operations:

    |Operation|Algorithm|
    |-|-|
    |get a block|getblk|
    |release a block|brelse|
    |read|bread|
    |Read Ahead|breada|
    |write|bwrite|

### 1) Getting a Block

**getblk( filesystem, block) : Locked buffer**

    while buffer not found:

        if buffer in Queue:
            if busy:
                sleep and continue
            return buffer

        else
            try to get a buffer from free-list
            if no buffer free:
                sleep and continue
            if delay-write status:
                write async
                sleep and continue
            else:
                return buffer
            

### 2) Releasing Block   
**brelse( A locked buffer): None**

    Wake up Process was put to  sleep for:
        This buffer
        Any Free buffer
    Increase execution level
    if buffer valid:
        put to tail of free-list
    else:
        put next to header
    decrease execution level
    remove lock

### 3) Reading Buffer
**bread( filesystem, block): Valid buffer**

    getblk 
    if valid:
        return buffer
    start disk read
    sleep for read
    return buffer

### 4) Reading buffer ahead:
**breada(filesystem, block 1, block 2): Valid buffer**
    
    if buffer 1 not in cache:
        getblk
        if invalid:
            disk read

    if buffer 2 not in cache:
        getblk
        if invalid:
            disk read
        buffer 2 brelse

    if buffer 1 in cache:
        bread
        return buffer 1
    sleep for buffer 1 disk read
    return buffer 1

### 5) Writing buffer to disk

**bwrite(buffer):None**

    start writing to disk
    if (synchronous):
        sleep for reading
        brelse buffer
    if (delayed write):
        mark delayed write
        add to free list head