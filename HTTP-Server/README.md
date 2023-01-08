# Multi-Threaded HTTP Server
Created in Spring 2022 for my Computer Systems Design class (CSE130). For this project I used C to create a multi-threaded HTTP server. In a previous class project, I started out by creating a simple HTTP server with methods GET, PUT, and a new method APPEND which as the name may describe, appends text to an existing file. Since we already created our own HTTP Server, the key part of this assignment was to  figure out how to pool threads and dispatch HTTP requests to our worker pool while ensuring atomicity and concurrency on our files. Our particular HTTP Server was hosted locally. A more in-depth look at the assignment can be seen in **Assignment.pdf**.

## Video Presentation
Basic HTTP Server functionality using `curl` to send requests
- [YouTube]()

## Description

### Modules:
When running my multi-threaded HTTP server, I have split it into three kinds of modules. A module for handling/parsing any requests, a module for handling/writing a response to the client (including the audit log), and a httpserver module for dispatching work amongst threads. 

In the parsing module, the requests that the clients send to the server are read in, and the type of request is interpreted. From there, the same module determines what needs to be done based on the given method (GET, PUT, or APPEND). For any GET requests, the server reads the request and tells the response module what file needs to be returned to the client. For any PUT or APPEND requests, the server reads the request and then writes the message body to the provided file as it continues to read. Then, it communicates to the response module a file descriptor for the written to file. After these methods send a file descriptor to the response module, the response to the client is handled.

If a GET response is picked in the response module, it reads in the given file descriptor, and upon success, it sends the client the contents of the file. For any PUT or APPEND request, the response module checks that the file descriptor was written to without any errors, otherwise it will return the kind of error that the file descriptor sent to `errno`.

The httpserver decides how many threads to create through the command line argument '-t'. When using this flag, specify the number of threads you want to run the server with. Afterward, the module will create that number of threads, and as requests come to the server, the module will queue and dispatch the requests to the threads. 

### Ensuring Atomicity and Coherency:
A big part of this final checkpoint for my HTTP server was to ensure atomicity and coherency, meaning that the requests will be processed as if they come one after another and that the files will represent the most recent request to them. In order to hold to these standards, I made sure to incorporate a reader and writer lock into my server. These locks mean that for any file, only 1 thread can access it if it is being written to. However, if the file is only being read to, any number of readers can access it at the same time. In order to keep the locks simple, I used the system function `flock` to manage my reader/writer locks. This function allows us to put an advisory lock on our URI files in the form of a shared lock or an exclusive lock. I used an exclusive lock for any writing operations (PUT/APPEND) since we want the URI file to be exclusive to one thread during those operations, and a shared lock on any reading operations (GET) since readers can share the file as they are not editing anything. 

Another key component was to write any requests to a temporary file before updating the actual URI file. One example where temporary files might be necessary is if a PUT request is sent, and then a GET request to the same file right after. These 2 requests might start being processed at the same time, and if the PUT is for a large file it might take a while to complete the request. However, GET will start reading the file right away, only reading part of the entire contents of the file since it isn't finished PUTing. With the use of a temporary file, the PUT request will write to that file instead of the real file, and once it is done PUTing, it will update the temporary file to be the real URI file. This way, the GET will only read the file entirely before the PUT has started, or entirely after the PUT is done, nowhere in between. 

### Data Structures: 
Buffers - While parsing any requests, the read data is stored in a buffer in blocks of 4096. This particular number doesn't matter too much, but if set too low it could affect efficiency. However, it is important to note that for our implementation the request line and headers will be at most 2048 bytes.

Linked List - When parsing my header fields, I used a linked list to store all of them since they are given in key-value pairs. For each node in my linked list, I can simply locate the node I want with a search for the key and access its value that way, so I felt that a linked list implementation would be quite useful since we could have an unknown number of header fields. The types of header fields I knew I would consistently need are Content-Length and Request-Id, so by using a Linked List lookup I can find their values quickly. 

Queue - In order to store the incoming requests, I decided to use a bounded circular queue. This particular queue stores the connection file descriptor that it receives when a request is sent to the server. It is important that this queue is bounded because if an enormous amount of requests comes in all at once, it is possible that the amount of data that those requests include is too much for the server to store at once (for example 10,000 10GB requests). Since the queue is bounded, it limits the amount that can be stored at once, meaning the server might not be overloaded. Once the connection file descriptors are queued, the dispatcher thread can simply dequeue a thread and send it off to the threads to work on.

### Types of Errors:

Some of the errors described in the design document were a bit on the vague side, so I thought I should describe them here. For this particular HTTP server implementation, the only errors we need to worry about are Not Found and Internal Server errors, otherwise we can assume that the request is given in the correct format (Not the case in previous assignments). In the case that there are no errors, either OK or Created will be returned to the client. It is important to note that any type of success or error is sent back to the client from the server in the form of a response.

Not Found: This error is mostly self-explanatory, it happens when we cannot find a file. This error can happen for any GET or APPEND request, however, a PUT request always creates a file if it is not found, so it avoids this error. 

Internal Server Error: This kind of error is quite vague, however from its definition it seems that it occurs when something unexpected happens. The only time I could think of something unexpected happening is when we are unexpectedly unable to read or write, and the read or write call returns -1.

### Data Races, Critical Regions, and Locks/Conditional Variables

Since we are using multiple threads to run our server, our server is prone to data races. This means we want to lock the places in which there are critical regions in our server (Places where it would be bad if the threads were editing it at the same time). The critical regions that I found were the bounded queue and files. 

For the bounded queue, if threads were adding and removing connections file descriptors from the queue at the same time it would be prone to data races, meaning we should lock out other threads when updating the queue. In order to stop these data races, I implemented a mutex lock around the enqueue/dequeue of an item in the queue. With this method, only one thread can edit the queue at a time. Another important feature I included was conditional variables. With conditional variables, the server is able to wait if the queue is full or empty, and signal the dispatcher and worker threads when there is more work to be done, or when the queue is no longer full. These conditional variables allow the server to not busy wait, which is quite important for saving CPU usage.  

For data races pertaining to files, I implemented the reader-writer lock. Since I have already described the reader-writer lock, it should be clear how a thread gets its lock with the `flock` function. However, the most important aspect of this lock is that only 1 thread can write to a URI at a time, and if only reading is being done to the URI, any number of threads can read.

### Audit Log:

The audit log consists of all the requests in the order the server processed them. This is particularly important for our multi-threaded server and keeping track of when each thread completes a request. 

**Usage:**
When starting the server, you can use the `-l` flag followed by a filename to tell the server to write to that file as the audit log. By default, the server will use `stderr` as the audit log. You will also need to ensure that any requests sent to the server include a Request-Id header with a number as that request's particular ID. This Request-Id header is how you will differentiate requests in the audit log. If no particular Request-Id is given, the default value will be 0. 
