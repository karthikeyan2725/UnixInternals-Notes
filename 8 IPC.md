## IPC
## 1) Pipe
### pipe(): reader Descriptor, writer Descriptor

    ialloc new inode
    create file table entry for reader, writer point to inode
    create descripter for reader, writer, point to file table entry
    set inode reference count = 2
    return reader,writer

* Each IPC contain 
     * table ipc entry with key
     * status
     * permission
     * recycled
     * get
     * index = desc mod total entry
    
### Messages:
* Messages as linked linked
[msg](images/ms)
* Message queue structure:
    * Total bytes on list
    * pointer to first and last message
    * maximum bytes
    * last process to send messages
    * time stamps of all operations
1)  msgget(key,flag): descriptor
#### 2) msgsnd(msg_descriptor, msg, count, flag):bytes sent
        check permissions
        while no space :
            if no wait flag:
                return
            sleep
        get a header
        copy data from user space to kernel
        modify ds:
                add header to queue
                header points data
                timestamps
                pid
        wake all process
#### 3) msgrecv(msg_des, buffer, size_buffer, msg_type, flag):
    
        check permission
        loop:
            if type == 0:
                select first message
            if type>0:
                select first message with given type
            if type<0:
                select message less than abs(type)
            if theres a message:
                adjust message size
                if too small error
                copy text from kernel to user
                unlink message from queue
                return
4) msgctl(id,cmd,mstatbuf)           

