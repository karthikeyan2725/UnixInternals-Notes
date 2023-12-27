## The overview of Unix System: 
`Nothing is better than a birds eye-view 🦅`
### The Story :
* **GE** and **MIT** joined together to develop multics, But it failed.
* Ken Thompson and Dennis Ritchie drew the plan for a new File system.
* Thompson programmed a kernel for it on a **GE467** machine.
* Brian Kernighan joined them.
* They created the first version of unix and ran it on **PDP-7**.
* They moved it to **PDP-11**.
* Thompson tried to implement the system on Fortran, but created B.
* B was slow, so Ritchie created **C**.
* The Unix was rewritten in C in 1973
* They published paper in 1974.
* More than 500 Unix websites in 1977.
* They created an amalgamation of all the version as the Unix - III
* Unix - V was created From UNIX - III

### Why the hell Unix was popular? 🤔
* Written using C
* A good file system 
* Everything in UNIX is treated as a file - Simplicity
* Fast processing speed.
* At the same time:
    * Multiple users can use it
    * Multiple process can run on it.

### System Structure:
 * UNIX abstracts the hardware to programs.
 * Program use system calls to communicate.
 * System Calls are also packaged into **commands**.
 * 32/64 System calls are used.
 * also a **CC** which gives **a.out** file.

### 👨Users Perspective on:
#### 1. The File System:
* Tree like Structure
* Root - "/"
* path name contains name and /.
* Traverse Using the cd command
* **.** : current directory
* **..** : parent directory
* directory is also an file, but *Special*.
* all internal nodes are directories
* External nodes (Leaf Nodes) can be a:
    * Directory File
    * Actual File
    * Device File
* cp used to copy files

#### 2. Processing:
* Programs converted into process executing some task.
* Many such process can run at the same time.
* There are calls to create, kill and synchronize them.
* The directory in which a process run is its environment.
#### 3. Shell
* Shell is an program that can execute **commands**.
* Commands are system calls packaged together.
* This package could be:
    * .exe
    * .sh
* The command is run by the syntax:

        [CommandName] [Options] [Parameters]
* When a command is run, the shell forks and exec the command's program file.
* They could be run synchronously or asynchronously to the shell process.
* Outputs of the commands can be:
    1) redirected (> and <) to files
    2) piped (|) between commands

### 🖥️ UNIX Perspective on: 
#### 1. The File System:
* Free Space management
* File Protection
#### 2. Processing:
* Powers of UNIX upon Processes:
    * Create 
    * Run
    * Sleep
    * Suspend
    * Kill
* The processes in main memory and Secondary Memory managed by UNIX.
* Schedule their execution.

#### 3. Shell:
* Some UNIX powers given to shell

### 💾 Hardwares Perspective on:
#### 1) Modes of execution:
* The hardware must follow the two modes of execution.
    * User
    * Kernel
* The process in User mode upgrades to Kernel mode when upon System Call.
* After Execution, they return to User Mode.
#### 2) Interrupt:
* The hardware has devices that may interrupt the UNIX.
* The UNIX pauses current process, services Interrupt and resumes the Process execution.
#### 3) Exception:
* Process executes instructions run by the hardware.
* Sometime they put the machine into a state of confusion, called Exception.
* They are handled by UNIX.
#### 4) Execution Levels:
* Interruption of important processes cannot be tolerated.
* Hence Processes are given levels to avoid critical process be stopped.
#### 5) Memory Management:
* Hardware must allow to storage of processes in Main memory and in Secondary Memory.

## The Kernel 🌽
### The Structure:
![Structure](images/unix%20structure.png)
* **Three** layer architecture.
* communication from User to Kernel level through system calls 
    * called by library function
    * called directly 
* system call interact with:
    * **File system** using:
        * open
        * close
        * read
        * write
        * chown
        * chmod 
    * **Process system** using:
        * fork
        * exec
        * exit
        * brk?
        * signal?
    * **Hardware implicitly** using:
        * File system
        * Process system
* These interactions are based on :
    * File system:
        * Buffer Cache
        * Device Drivers
    * Process System:
        * Scheduling
        * Memory Management
        * IPC
    * Hardware:
        * Interrupt handling

### System Concepts:
### 1) 📁 The File Subsystem:
#### Perspective of Main Memory:
* There are three structures
    * Inode table
    * File Table
    * User File Descriptor Table

* The disk Inode structure is loaded into main memory. 
* Inode table contains pointer to these in-memory copies.
* The File table contains pointer to inodes.
* Descriptor contains pointer to file table entries.
* Processes can use this table to access and modify files.

* NOTE: The **root inode ("/")** of filesystem is placed into memory when mounting.

#### Perspective of Disk:
* The disk can be divided into many file systems.
* Each filesystem contain:
    ![file system](images/file%20system.png)
* **Boot block** contain code to load the OS into memory.
* Not all file system has this boot block.
* **Super block** contains free space locations.
* **Inode List** for files.
* **Data block** of files.

### 2) 🏃‍♂️ Process Subsystem:
### Process's Perspective:
#### Process Necessities:
* Process needs memory for :
    * Text - Machine code
    * Data - to store variable
    * Stack  - for function call
* Regions of memory given to the process, to store them.
#### Relation between process:
* All process in the memory come from **forking** and **exec**.
* A process calling fork is parent and the new process is child.
* The exception to this is the **process 0**
#### Modes of Process:
* To avoid process in user mode gaining kernel level information, seperate stacks for each mode.

    |Stack Frame|
    |-|
    |Parameters|
    |Return Address|
    |Local Variables|
    |Address of previous Frame|
![Kernel Vs User Stack](images/kernel%20vs%20user%20stack.png)
### Memory's Perspective:
#### Data Structures Involved:
* Four Data Structures:
    * Region Table
    * pregion Table
    * Process Table
    * U area

![Process subsystem](images/process%20system.png)   

* Regions are block of memory.
* Region can be:
    * Text
    * Data
    * Stack
* **Region Table** has region's location and type.
* **Pregion Table** points to region table.
* **process table** points to pregion table.
* **U area** points to process table

* U area also contains information of process such as:
    * Process size
    * Pointer to File Descriptor 
    * Current Directory
    * Current Root
* Process Table contains:
    * State Field
    * UID
    * Event Descriptor for suspension.
NOTE: Root changes based on file system

### Context of Process:
All the activity of process:
* values in CPU register (PC, IR, PS,...)
* u area of the process
* user and kernel stack
* global variables 

### Process States
* Ready
* Executing in Kernel Mode
* Executing in User Mode
* Sleep

### Process Transistions
* The diagram is self-explanatory:

![Process States](images/Process%20States.png)

### Sleep and wakeup
* Process put to sleep when process needs a event to ocuur.
* If event occurs then the process is woke up.

### 3) Kernel Data structure:
* All of the before said data structures are fixed size.
* There is a possiblility running out of entries.
* UNIX does not allow this problem, thanks to the way it's structured.

### 4) Administration With UNIX:
* Administration include:
    * creation of user and groups for different people.
    * Installation of programs.
    * File Permissions
    * Disk formatting

    etc...
* From UNIX perspective, they are user processes.
* A **Super-User** has priveleges to do these tasks.