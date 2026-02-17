Linux-specific API for asynchronous I/O. It allows the user to submit one or more I/O requests, which are processed asynchronously without blocking the calling process.

## Concurrency architecture
### **Iterative**: 
- This serves one request after another. 
- While it is serving one request, other requests that might arrive have to wait till the previous one is done processing. 
- There is [a limit](http://man7.org/linux/man-pages/man2/listen.2.html) to how many requests the operating system will queue up.
	- By default, Linux queues up to 128 for kernel versions below 5.4 and 4,096 in newer ones.
    
### **Forking**: 
- This creates a new process for each request that needs to be served. 
- Requests don’t have to wait for previous requests to get processed. 

### **Preforked**: 
- This avoids the overhead of having to create a totally new process every time a request needs to be served. 
- It creates a pool of processes that are assigned requests as they come in. Only when all the processes in the pool are busy should incoming requests have to wait for their turn to get processed. 
- Can tweak the number of processes in the pool depending on the load they usually experience.

### **Threaded**: 
- This spawns a new thread every time a request needs to be processed. 
- Threads share a lot of data with the main process creating it and thus incur a slightly lower overhead during creation compared to creating a new process [1](https://unixism.net/loti/async_intro.html#id2).

### **Prethreaded**: 
- This is the threads equivalent of the `preforked` architecture. 
- In this style, a pool of threads are created and threads from the pool are assigned requests as they come in. 
- As in the preforked model, requests have to wait only if all threads are busy processing previously received requests.
- A very efficient model and is the one followed by most web application frameworks.

### **Poll**: 
- This single threaded and uses the [poll(2)](http://man7.org/linux/man-pages/man2/poll.2.html) system call to multiplex between requests. 
- [poll(2)](http://man7.org/linux/man-pages/man2/poll.2.html) however has performance problems scaling to a large number of file descriptors.
- The state for each request is tracked and a series of callbacks to functions are made that take processing of that request to the next stage.

### **epoll** 
- This is a single-threaded, server that uses the [epoll(7)](http://man7.org/linux/man-pages/man7/epoll.7.html) family of system calls in place of [poll(2)](http://man7.org/linux/man-pages/man2/poll.2.html), but is otherwise, architecturally the same.

![[Request-vs-Concurrency.png]]

Thread pool based programs are _way_ easier to code compared to their asynchronous counterparts.

## Model
Set up shared buffers with:
- [io_uring_setup(2)](https://man7.org/linux/man-pages/man2/io_uring_setup.2.html) 
- [mmap(2)](https://man7.org/linux/man-pages/man2/mmap.2.html)
Mapping into user space shared buffers for the submission queue (SQ) and the completion queue (CQ).
- Place I/O requests on the `SQ`
- Kernel places the results of the operations on `CQ`

For each I/O request (read a file, write a file, accept a socket connection, ...) create `submission request (SQE)`, describe the I/O operation and place it at the tail of the `SQ`

After you add one or more `SQEs`, you need to call [io_uring_enter(2)](https://man7.org/linux/man-pages/man2/io_uring_enter.2.html) to tell the kernel to dequeue your I/O requests off the `SQ` and begin processing them.

 The kernel places exactly one matching `CQE` in the `CQ` for every `SQE` you submit on the `SQ`.
 - Read `CQEs` off the head of the `CQ` ring buffer
 - The `res` field of the `CQE` corresponds to the return value of the system call equivalent
 - Request completions are not in the order of placement in `SQ`

[io_uring_enter(2)](https://man7.org/linux/man-pages/man2/io_uring_enter.2.html) can also wait for specified # of requests  to be processed by the kernel before it returns.

When more than one request is in flight, it is not possible to determine which one will execute or  complete first.  When you dequeue `CQEs` off the `CQ`, you should always check which submitted request it corresponds to.  
- Utilizing the _`user_data`_ field in the request, which is passed back on the completion side.

Concretely, for operations where strict ordering is required, such as for sends and receives on a stream-oriented TCP socket: It is generally unsafe to have more than one outstanding send, or more than one outstanding receive (_the two directions are independent_) on a given socket at a time.

**IO_uring** provides various facilities to enable applications to efficiently pipeline their operations
- Applications submitted in a single batch may use`IOSQE_IO_LINK` to enforce execution order in the kernel.
- `multi shot` and send/receive `bundles` to allow applications to provide more data in fewer, more efficient trips to the kernel.
Applications must still ensure they do not overlap different sends or different receives on a given file.

Add to & reading from the queues:
1. Add `SQEs` to the **tail** of the `SQ`. The kernel reads `SQEs` off the **head** of the queue.
2. Kernel adds `CQEs` to the **tail** of the `CQ`.  You read `CQEs` off the **head** of the queue.

**IOSQE_CQE_SKIP_SUCCESS** - skips posting `CQE` for every `SQE`

## SQ Polling
**Polling mode** avoids call to *`io_uring_enter`* which you use to inform the kernel that you have the `SQEs` on to the `SQ`.

`SQ Polling` starts a a kernel thread that polls the submission queue for any IO requests you submit by adding SQEs.  With it enabled,  designated kernel thread dequeues `SQEs` off the SQ as you add them and dispatches them for asynchronous processing.

## Setting up io_uring
These ring buffers are shared between kernel and user space. 
Set up with [`io_uring_setup()`](https://unixism.net/loti/ref-iouring/io_uring_setup.html#c.io_uring_setup "io_uring_setup") and then mapping them into user space with 2 [mmap(2)](https://man7.org/linux/man-pages/man2/mmap.2.html) calls.

```c
#include <linux/io_uring.h>
#include <sys/mman.h>

int io_uring_setup(u32 entries, struct io_uring_params *p)
void *mmap(size_t length; void addr[length], size_t length, int prot, int flags, int fd, off_t offset);
```

 [io_uring_setup()](https://unixism.net/loti/ref-iouring/io_uring_setup.html#c.io_uring_setup) - system call sets up a `SQ` and `CQ` with at least <font color=#30D5C8>entries</font> entries, and returns a file descriptor which can be used to perform subsequent operations on the io_uring instance.
- <font color=#30D5C8>params</font> is used by the application to pass options to the kernel, and by the kernel to convey information about the ring buffers.
```c
struct io_uring_params {
    __u32 sq_entries;
    __u32 cq_entries;
    __u32 flags;
    __u32 sq_thread_cpu;
    __u32 sq_thread_idle;
    __u32 features;
    __u32 resv[4];
    struct io_sqring_offsets sq_off;
    struct io_cqring_offsets cq_off;
};
```

- The _flags_, <font color=#30D5C8>sq_thread_cpu</font>, and <font color=#30D5C8>sq_thread_idle</font> fields are used to configure the io_uring instance. 
- <font color=#30D5C8>flags</font> is a bit mask of 0 or more of the following values ORed together:

 `mmap()` - creates a new mapping in the virtual address space of the calling process.  The starting address for the new mapping is specified in _addr_.  
- The _length_ argument specifies the length of the mapping (which must be >0).
- If _addr_ is NULL, then the kernel chooses the (page-aligned) address at which to create the mapping; this is the most portable method of creating a new mapping.
- File mapping contents are initialized using <font color=#30D5C8>length</font> bytes starting at offset <font color=#30D5C8>offset</font> in the file (or other object) referred to by the file descriptor <font color=#30D5C8>fd</font>.  
- <font color=#30D5C8>Offset</font> must be a multiple of the page size as returned by <font color=#ffcba4>_sysconf(_SC_PAGE_SIZE)_.</font>

Tell `io_uring` what's to be done (read or write a file, accept client connections, etc), which you describe as part of a `SQE` and add it to the tail of the `SQ`.
- Tell the kernel via the [`io_uring_enter()`](https://unixism.net/loti/ref-iouring/io_uring_enter.html#c.io_uring_enter "io_uring_enter") system call that you’ve added an `SQE` to the submission queue ring buffer. 
- You can add multiple `SQEs` before making the system call as well.
-  [`io_uring_enter()`](https://unixism.net/loti/ref-iouring/io_uring_enter.html#c.io_uring_enter "io_uring_enter") can also wait for a number of requests to be processed by the kernel before it returns so you know you’re ready to read off the completion queue for results.

Adding submission queue entries to a ring buffer and reading completion queue entries off of the completion queue ring buffer.
### readv() & writev()
Part of the set of **System calls** that enable *scatter/gather* I/O  also known as <font color=#ffcba4>vectored I/O</font>. 
Arguments: 
1. File descriptor
2. Pointer to an array of <font color=#ffcba4>struct iovec</font> structures
3. length of that array
```c
// Each struct simply points to a buffer. 
// A base address and a length.
struct iovec{
	void *iov_base; /* Starting address */
	size_t iov_len; /* Number of bytes to transfer */
};
```

Why use vectored or scatter/gather I/O over regular `read()` and `write()` :
1. Fill in many members of a structure without either resorting to copying around buffers or making multiple calls to `read() / write()`
2. Calls are atomic while multiple calls to `read()` and `write()` are not

### CQE
An instance of a struct which the kernel responds with for every SQE struct instance that is added to the submission queue. This contains the results of the operation you requested via an SQE instance.

```c
struct io_uring_cqe {
	__u64  user_data;  /* sqe->user_data passed back */
	__s32  res;    /* result code for this event */
	__u32  flags;
};
```

`CQEs` can arrive in any order as they become available. Whichever operation finishes quickly, it is immediately made available.

To identify the which SQE request a particular CQE:
- *`user_data`* field to identify it by passing a pointer.

`CQES` corresponds with a system calls return value store in *`res`*:
- read operation: # of bytes read / -errno if there's an error.

### SQE
More complex as the entries have to be generic enough to represent wide range of IO operations

```c
struct io_uring_sqe {
	__u8  opcode;    /* type of operation for this sqe */
	__u8  flags;    /* IO-SQE_ flags */
	__u16  ioprio;    /* ioprio for the request */
	__s32  fd;    /* file descriptor to do IO on */
	__u64  off;    /* offset into file */
	__u64  addr;    /* pointer to buffer or iovecs */
	__u32  len;    /* buffer size or number of iovecs */
	union {
	    __kernel_rwf_t  rw_flags;
	    __u32    fsync_flags;
	    __u16    poll_events;
	    __u32    sync_range_flags;
	    __u32    msg_flags;
	};
	__u64  user_data;  /* data to be passed back at completion time */
	union {
	    __u16  buf_index;  /* index into fixed buffers, if used */
		__u64  __pad2[3];
	};
};
```

- `opcode` - specify the operation, in case, `readv()` using the `IORING_OP_READV` constant.
- `fd` is used to specify the file which we want to read from
- `addr` is used to point to the array of `iovec` structures that hold the addresses and lengths of the buffers we’ve allocated for I/O.
- `len` is used to hold the length of the arrays of `iovecs`.



## IO_Uring vs readV() file read & console concatenation
### Test with console rendering:
```txt
>>> STARTING READV TEST
--- Start of Block 0 ---
...

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           1.225 ms
Throughput:           18.66 MiB/s
========================================

>>> STARTING IO_URING TEST
--- Start of Block 0 ---
...

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.737 ms
Throughput:           31.05 MiB/s
========================================
```

The *Console Bottleneck* the throughput was only **18–31 MiB/s**.

<font color=#ff0800>The reason :</font> Printing to the console (`fputc`) is incredibly slow compared to memory or disk I/O. Your CPU spent 95% of its time waiting for the terminal to render text.


### Test without console rendering
```txt
>>> STARTING READV TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.058 ms
Throughput:          394.18 MiB/s
========================================

>>> STARTING IO_URING TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.027 ms
Throughput:          842.25 MiB/s
========================================
```

Analysis:
- **`readv()`:** **394 MiB/s**
- **`io_uring`:** **842 MiB/s**

This is because `readv()` is a **blocking system call**. Every time you call it, the CPU has to <font color=#ffcba4>context-switch :</font> it pauses your program, jumps into kernel mode, does the work, and jumps back.

With `io_uring`, since we've mapped the rings into your own memory space, the overhead of "talking" to the kernel is significantly lower. 

---

### A Final Check on the Metrics

**Total Time** for the `io_uring` quiet run: **0.027 ms** / **27 microseconds**.

To put that in perspective:
- A human blink takes about **100–400 ms**.
- `io_uring` sets up a ring, allocated memory, performed a scatter-gather read of a file, reaped the completion, and cleaned up the memory in **1/10,000th of a blink**.

### Batch submissions
Currently, the code waits for the kernel to finish every single file before moving to the next one. By batching, we fill the "pipeline," allowing the kernel to handle multiple disk requests simultaneously.

#### The Batching Strategy
1. **Submit All:** Loop through all files and add them to the Submission Queue (`SQ`) without calling `io_uring_enter` with the `GETEVENTS` flag yet.
2. **Enter Once:** Call `io_uring_enter` once to tell the kernel "here are $N$ tasks."
3. **Reap All:** Loop through the Completion Queue (CQ) until we've processed all $N$ results.


#### 1. Refactored `submit_to_sq`
Remove the `io_uring_enter` call from inside this function so it only _prepares_ the request.

#### 2. Refactored `uring_cat` (The Orchestrator)
Logic changes to a "Two-Pass" system.
```c++
unsigned get_sq_occupancy(struct submitter *s) { 
	// Current tail minus current head = number of entries the kernel hasn't processed yet 
	return *s->sq_ring.tail - *s->sq_ring.head; 
}

//---------------------------//

int files_to_process = 0;
int files_completed = 0;
size_t total_bytes = 0;

// PASS 1: Submit everything
for (int i = optind; i < argc; i++){
	// --- SPACE CHECK ---
    // If the SQ is full, we MUST reap at least one completion to make space
    while (get_sq_occupancy(s.get()) >= QUEUE_DEPTH){
        // Tell kernel to process what's there and wait for 1 completion
        io_uring_enter(s->ring_fd, 0, 1, IORING_ENTER_GETEVENTS);
        files_completed += read_from_cq(s.get(), quiet);
    }

    ssize_t bytes_read = submit_to_sq(argv[i], s.get());
    if (bytes_read < 0){
        std::println(stderr, "Error reading file: {}", argv[i]);
        return 1;
    }
    total_bytes += bytes_read;
    files_to_process++;
}

/*
* TELL THE KERNEL: "Go! And don't come back until at least 1 is done."
* Submit 'files_to_process' and wait for at least 1.
* Passing in the IOURING_ENTER_GETEVENTS flag which causes the
* io_uring_enter() call to wait until min_complete events (3rd param) complete.
*/
io_uring_enter(s->ring_fd, files_to_process, 1, IORING_ENTER_GETEVENTS);

// PASS 2: Reap everything
while (completed < files_to_process){
    // This will now process all available CQEs in the ring
    files_completed += read_from_cq(s.get(), quiet);

    // If we haven't finished yet but the queue is empty,
    // wait for more events.
    if (completed < files_to_process){
        io_uring_enter(s->ring_fd, 0, 1, IORING_ENTER_GETEVENTS);
    }
}
```


#### readv() vs io_uring vs io_uring batch processing
1. Single file reads
```txt
>>> STARTING READV TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.105 ms
Throughput:          217.21 MiB/s
========================================

>>> STARTING IO_URING TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.134 ms
Throughput:          171.10 MiB/s
========================================

>>> STARTING IO_URING BATCH-PROCESSING TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.02 MiB
Total Time:           0.028 ms
Throughput:          820.07 MiB/s
========================================
```

2. Multiple file reads (10 files)
```txt
>>> STARTING READV TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.08 MiB
Total Time:           0.174 ms
Throughput:          434.18 MiB/s
========================================

>>> STARTING IO_URING TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.08 MiB
Total Time:           0.186 ms
Throughput:          406.11 MiB/s
========================================

>>> STARTING IO_URING BATCH-PROCESSING TEST

==============================
     PERFORMANCE METRICS
==============================

========================================
         I/O PERFORMANCE REPORT
========================================
Total Data:            0.08 MiB
Total Time:           0.687 ms
Throughput:          110.15 MiB/s
========================================
```

#### Why your `single read` was faster than `multple-read` :

In `single read`, you had 1 file. The overhead of 1 `open()` and 1 `fstat()` is negligible. In `multple-read`, if you had n files, you incurred that overhead n times. Because the files are tiny (0.02 MiB), the **<font color=#ff2800>time to open  > time to read</font>.**

To bypass the <font color=#30D5C8>Open Bottleneck</font>. we'll use `IORING_OP_OPENAT` so that even opening the files happens asynchronously inside the ring.

### Batch submissions & Asynchronous file reads
The submission loop becomes `non-blocking`: tells the kernel "Open this, then read it," and the CPU moves to the next file immediately without waiting for the filesystem metadata.

To achieve this, we use a feature called **SQE Chaining** (`IOSQE_IO_LINK`). This tells `io_uring`: "Don't start the Read until the Open succeeds."

### The New Architecture: Linked Operations

We will now submit **two** SQEs per file:

1. **SQE 1:** `IORING_OP_OPENAT` (Opens the file and stores the FD in the kernel).
2. **SQE 2:** `IORING_OP_READV` (Linked to the first; starts as soon as the FD is ready).


Update your `file_info` struct to hold the `dir_fd` (usually `AT_FDCWD`) and the `file_path`.
```c++
ssize_t submit_to_sq_async(char *const file_path, struct submitter *s) {
    struct app_io_sq_ring *sring = &s->sq_ring;
    
    // We need 2 slots in the ring for one file (Open + Read)
    if (get_sq_occupancy(s) > QUEUE_DEPTH - 2) return -1; 

    auto fi = std::make_unique<file_info>();
    // ... calculate blocks/iovecs as before, but DON'T open() yet ...

    unsigned head_idx, open_idx, read_idx;
    unsigned tail = *sring->tail;

    // --- 1. THE OPEN SQE ---
    open_idx = tail & *s->sq_ring.ring_mask;
    struct io_uring_sqe *sqe_open = &s->sqes[open_idx];
    sqe_open->opcode = IORING_OP_OPENAT;
    sqe_open->fd = AT_FDCWD;
    sqe_open->addr = reinterpret_cast<uintptr_t>(file_path);
    sqe_open->open_flags = O_RDONLY;
    sqe_open->flags = IOSQE_IO_LINK; // <--- LINK: Read depends on this!
    sqe_open->user_data = 0; // We don't necessarily need a callback for the open itself

    // --- 2. THE READV SQE ---
    read_idx = (tail + 1) & *s->sq_ring.ring_mask;
    struct io_uring_sqe *sqe_read = &s->sqes[read_idx];
    sqe_read->opcode = IORING_OP_READV;
    sqe_read->fd = -1; // The kernel will use the result of the previous linked SQE!
    sqe_read->flags = 0; 
    sqe_read->addr = reinterpret_cast<uintptr_t>(fi->iovecs.data());
    sqe_read->len = fi->get_blocks();
    sqe_read->off = 0;
    
    // Ownership handoff
    sqe_read->user_data = reinterpret_cast<uintptr_t>(fi.release());

    // Update tail by 2
    write_barrier();
    *sring->tail = tail + 2;

    return 0; 
}
```

- **Zero Blocking:** The `for` loop runs at the speed of RAM. It just writes 128 bytes (two SQEs) to the ring and moves on.
    
- **Kernel Concurrency:** The kernel can have 10 threads opening 10 different files on the disk controller while your program is still submitting file #11.
    
- **Pipeline Efficiency:** eliminate the "System Call Gap" where the CPU sits idle waiting for the disk head to find the file metadata.

```c++
size_t total_bytes_reaped = 0;

int read_from_cq(struct submitter *s, bool quiet = false){
        struct app_io_cq_ring *cring = &s->cq_ring;
        unsigned head = *cring->head;
        int completed_files = 0;

        while (head != *cring->tail)
        {
            read_barrier();
            struct io_uring_cqe *cqe = &cring->cqes[head & *cring->ring_mask];

            // Distinguish between OPENAT and READV
            if (cqe->user_data == 0)
            {
                // OPENAT completion
                if (cqe->res < 0)
                {
                    std::println(stderr, "Async Open Error: {}", strerror(abs(cqe->res)));
                }
            }
            else
            {
                // READV completion
                std::unique_ptr<file_info> fi(reinterpret_cast<file_info *>(static_cast<uintptr_t>(cqe->user_data)));

                if (cqe->res < 0)
                {
                    std::println(stderr, "Async Read Error: {}", strerror(abs(cqe->res)));
                }
                else
                {
                    total_bytes_reaped += cqe->res;
                    if (!quiet)
                    {
                        for (auto const &iov : fi->iovecs)
                        {
                            output_to_console(static_cast<char *>(iov.iov_base), iov.iov_len);
                        }
                    }
                }
                completed_files++; // One whole file lifecycle finished
            }

            head++;
        }

        write_barrier();
        *cring->head = head;
        return completed_files;
}
```

- **Multiple CQEs:** receiving **two** CQEs per file. One for the `OPENAT` and one for the `READV`.

- **The "FD Summing" :** Currently, `total_bytes_reaped += cqe->res` will add the **File Descriptor number** (e.g., 3, 4, 5) to throughput total because the `OPENAT` completion returns the FD in the `res` field.

- **RAII Cleanup:** Only the `READV` CQE carries your `file_info` pointer. If you try to cast `user_data` to a `unique_ptr` on the `OPENAT` completion, it'll crash (since set to `0`).

Implied difference to batch processing:
- **CPU Parallelism:** submission loop (Pass 1) will now finish in microseconds because it never touches the disk.
- **Kernel Efficiency:** The kernel's `io_uring` worker threads will handle the `open()` and `read()` in the background while the main thread is already prepping the next file.

