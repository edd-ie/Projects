#Network #research #Cpp 

## Types of sockets

**Stream Sockets (SOCK_STREAM):** provide two-way,  reliable, and sequenced communication channels that operate over Transmission Control Protocol (TCP). 
- Ensure data integrity and order for applications where *delivery order and data integrity* are paramount.

**Datagram Sockets (SOCK_DGRAM):** facilitate communication, enabling connectionless messaging where each message is an independent packet using User Datagram Protocol (UDP). 
- Do not guarantee order or reliability of messages, advantageous for applications that *require speed and efficiency* 

**Sequenced Packet Sockets (SOCK_SEQPACKET):** Combining features from both SOCK_STREAM and SOCK_DGRAM, they provide a neutral ground offering reliable packet sequencing through a connection oriented service.

---
## Lifecycle
### Creation
Essential libraries:
`<sys/socket.h>` - socket calls
`<netinet/in.h>` - handling internet addresses. 
`arpa/inet.h>` - IP address functions
`unistd.h>` - system calls, respectively.

Created using the `socket()` system call, which requires:
1. The protocol family (usually `AF_INET` for IPv4 or `AF_INET6` for IPv6).
2. The socket type. (`SOCK_STREAM`, `SOCK_DGRAM`, `SOCK_SQPACKET`)
3. The protocol. (0 = OS selects based on the domain and type)
```c++
#include <sys/socket.h> 
#include <netinet/in.h> 
#include <arpa/inet.h> 
#include <unistd.h>
 
int socket_fd = socket(AF_INET, SOCK_STREAM, 0); 
if (socket_fd < 0) { 
	perror("Error creating socket"); 
} 
```

### Binding
Associating a socket with a specific port and IP address on the local machine using the `bind()` call. 
- Assigns a specific communication endpoint to the socket within the local network interface, making it recognizable when data arrives.
- Params: socket descriptor, pointer to addr structure, size of addr structure
```c++
struct sockaddr_in server_address; 
memset(&server_addr, 0, sizeof(server_addr));

server_address.sin_family = AF_INET; 
server_address.sin_port = htons(8080); 
server_address.sin_addr.s_addr = INADDR_ANY;
 
if (bind(socket_fd, (struct sockaddr*)&server_address, 
sizeof(server_address)) < 0) { 
	perror("Binding failed"); 
}
```

### Listening
The socket is set to accept incoming connections. The `listen()` function sets the maximum number of queued connections: 
```c++
if (listen(socket_fd, 5) < 0) { 
	perror("Error on listen"); 
} 
```
- **Backlog Queue**: The second parameter to listen defines the number of active, unaccepted connections that can be queued, waiting for service. 
	- `SOMAXCONN` - constant allows the system to manage the queue size dynamically. 
- **Passive Role:** - Listen mode transforms the socket into a passive one, incapable of initiating data exchange until a connection is accepted. 
	- Crucial for handling incoming client-server dialogues effectively, preventing premature shutdowns or excessive resource utilization.

### Accepting Connections
A socket is instantiated for each client connection, created as a result of the `accept())` system call. 
- Provides a dedicated communication channel
```c++
struct sockaddr_in client_addr; 
socklen_t client_length = sizeof(client_addr);

int client_socket = accept(socket_fd, (struct sockaddr*)&client_address, &client_length); 

if (client_socket < 0) { 
	perror("Accept failed"); 
} 
```

### Data Transmission
Data packets are sent and received via sockets, using `send()` and `recv()`
```c++
int n = send(client_socket, buffer, sizeof(buffer), 0); 
if (n < 0) { 
	perror("Error sending data"); 
}

n = recv(client_socket, buffer, sizeof(buffer), 0); 
if (n < 0) { 
	perror("Error receiving data"); 
}
```

### Termination:
The socket is closed once communication is complete, freeing resources:
```c++
close(socket_fd); 
close(client_socket);
```

### Additional concepts
1. socket options - can be configured using the `setsockopt()` function, options like
	- `SO_REUSEADDR` -  binding a socket to an address that may be presently in use, avoiding common startup delays
	- `SO_LINGER` - how long the socket remains in an active state as it finishes sending pending data.
	- `TCP_NODELAY`, 
	- These options alter how data is buffered, how connections are reused, and the handling of unfinished communications.
2. **Concurrency Control:** Because sockets are a shared resource and communications occur asynchronously, 
3. **Network Byte Order:** sometimes require consistency in data representation. To facilitate this, conversion functions like `htonl()` (host to network long), `ntohl()` (network to host long) ensure correct byte order across different architectures. 
	- important when exchanging binary data between heterogeneous systems.
4. **Non-blocking and Blocking IO:** default, sockets operate in blocking mode.
	- `fcntl` - can toggle the socket into non-blocking mode

```c++
#include <fcntl.h> 
int flags = fcntl(sockfd, F_GETFL, 0); 
fcntl(sockfd, F_SETFL, flags | O_NONBLOCK);
```
To effectively manage multiple socket requests, IO
multiplexing approaches like #I/O :
- select, poll, or epoll 
Help to respond dynamically to varying network demands. Allow a single thread to monitor numerous sockets,
adapting as data read or write requirements evolve.

### Concurrent Server
#### Threads
**Thread Creation and Management:** - Each connection spawns a new thread that manages client-specific interactions, thereby decoupling connection logic from the main thread and augmenting scalability through parallel processing. 

**Thread Detachment:** - The `pthread_detach` ensures that resources are properly freed once threads have completed execution, preventing memory leaks or undue resource accumulation

```c++
#include <pthread.h> 
void* handle_client(void* arg) { 
	int client_sockfd = *((int*)arg); 
	char buffer[1024]; 
	// Communicate with the client 
	recv(client_sockfd, buffer, sizeof(buffer), 0); 
	cout << "Client message: " << buffer << endl; 
	// Echo message back to the client 
	send(client_sockfd, buffer, sizeof(buffer), 0); 
	// Client connection ending 
	close(client_sockfd); 
	return NULL; 
} 
pthread_t client_thread; 

// Inside the main accept loop 
int client_sockfd = accept(sockfd, (struct 
sockaddr*)&client_addr, &client_len);
 
if (pthread_create(&client_thread, NULL, handle_client, 
(void*)&client_sockfd) != 0) { 
	cerr << "Failed to spawn thread" << endl; 
	close(client_sockfd); 
} 
pthread_detach(client_thread);
```
