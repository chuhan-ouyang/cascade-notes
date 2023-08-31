Fuse Client Low Level
* Launching point
* Initial steps: preparing configs, initialize fuse sessions, handling CLI
* Start a fuse client context type (fuse filesystem context)
Create a new sessions: two key arguments: fs_ops and fcc
  * Throw Exceptions if cannot register signal handlers or mount sessions
  * This session can further be used for mounting and the user loop
* Fcc corresponds to the fuse client context, corresponding the the use data specified by fuse
* Fs_ops is core of the logic, maping which function to execute upon Fuse operations
    * The registered handle function are called with the libfuse receives appropriate functionality from the user's requests
* Design pattern: overarching handler + doMethod to actually perform the handler's work
* Notable fs_ops: parameters are usually fuse_req_t for requst data, inode, and the file information (file handler, info about an open file for file descriptors)
* init
  * Called when libfuse estalishes communication with the FUSE kernel module
  * populate inodes based on json files
* destroy
* lookup
  * get the directory entry information of this inode
  method provided in the fuse client context module
  traversal based on whether the inode is designated as root or not
  * return directory information for the name, metadata inode, object pool inode, and admin metadata inode
* attributes (get and set)
    * set attributes based on the specific inode type
    * Then reply with the attributes requests 
* mk node
  * Currently does not support?
* make directories
  * Currently does not support?
* open
  * check flags for permission, reply with error if does not have permision
  * invoke the open file method in fuse client context 
* read
    * Read into a file bytes object and reply with it
    * File bytes is simply a stream of bytes
* write
  * Similar to read
  * Invoke the write file method in context and reply
  * Question: how does the read_file, write_file in the fuse client inode class work?
* release
    * close file and release the resources in kernel
* readdir
  * simillar to mkdir
* create


Fuse Client HL: Main difference, instead of working with fuse defined user data structure, work directly with string file paths
* Interface with fuse hl interface, mainly the fuse_operations struct
    * Fuse_file_info struct: retain infromation about an open file. File handlers created by open, opendir, and create and released by release, releasedr methods
      * Flags, cache, flush, seek status and file handle id
    * stat struct
      * device, file link count, user id, group id, file size, block size, metadata about modification times
* Main execution flow
  * Pass in the arguments, init fuse obj, command line options for threading model
  * Init a new fuse session, with the defined cascade_fs_oper (code of the code)
  * Use the new session for mounting to a directory, and running the loop (continuously handle client FS requests )
  * If there are exceptions, correspondingly unmount, destory, and free
* Core code: cascade_fs_oper: map the fuse operations to defined handlers

Handlers: Mostly take the parameters as the path (string), buffer, file into, flags, and function pointers
* Mostly, each method here correspond an internal method on the node defined in fcc_hl
* get attr
  * obtain the current node based on context 
  * write data into the buffer of the current node's context 
  * obtain status data based on context's ids, current time, and the flags 
* mkdir, rmdir
  * check if existed already
  * obtain the object pool root
  * Add snapshot if it is the root 
  * Then, with the op root existing already, set the path to be a corresponding node 
  * Find the appropriate pointer and insert into the parent-child trree structure
* chmod, chown
  * not implemented yet
* open, create
  * use the tree structure to get to the corresponding node
  * if valid fd, then set the file handle id of the current file info object to the found node
* read
  * search for the node base don the provided stirng path
  * access the bytes through the node 
  * copy the bytes into the provided buffer
* write
  * search for the node based on the provided string path
  * raise exceptions if no write permissions
  * resize the bytes of the node for the incoming write data
  * memcopy
* release 
  * search for the node based on the path
  * put the node into ServiceClientAPI
  * emplace into Blob?
  * ser versions to invalid
* Many methods use the fcc() method: calls the fuse get current context, and 

Key Data Structures: Fuse Context, Fuse Client Context, Fuse Object, Private Data, Path Trees

Struct Fuse Context
* Holds pointer to the fuse object


Fuse Object (with private data)


Fuse Client Context
* Inodes definitions
* Inodes: population, open, read, write
* Dispatching logic for a general request passed from the Fuse Client to be processed within a specific inode context 

Path Tree
* Offer methods to traverse the treee, find the root, search the node structure based a on a string path, create/insert node, 

 