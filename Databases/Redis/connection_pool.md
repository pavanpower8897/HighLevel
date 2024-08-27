https://medium.com/geekculture/the-pooling-of-connections-in-redis-e8188335bf64

Connection pooling is a technique where connections will be created at the begining instead of creating each whenever required. So we can simply reuse the existing connection (Borrow and give it back once work done)

So whenever an

- Whenever EC2 Instance initializes we try to establish connections with all redis nodes with certain configuration like number of connection, maxIdletime etc..
- Redis node server at OS level whenever client connection is requested it creates a network socket.
- In redis there is an event loop(Event driven architecture) which runs always and contains the information of incoming connection requests and connected network socket ids , So this event loop listen to incoming requests on these registered network sockets and pushes each request to queue for processing it. This is how redis handles the requests in single threaded mode.

If all connections are busy in executing the commands, If new commands comes in then depending on the client redis library we are using things will be handled accordingly like

- Blocking : Some client libraries and connection pooling blocks the commands and wait until one of the connection becomes free.
- Timeout or Exception: Wait for some timeout for connection getting free else timeout or throws exception.
- Connection Pool Expansion

Pros:

- Reduces the overhead of establishing connections again and again.

Here's a high-level overview of how connections are managed at the OS layer in Redis:

1. Network Socket Creation: When a client application initiates a connection to the Redis server, the OS creates a network socket to facilitate communication between the client and the server. The socket represents an endpoint for network communication.
2. TCP/IP Communication: Redis uses the TCP/IP protocol for network communication. The OS handles the transmission of data between the client and the server over this TCP/IP connection. It ensures reliable and ordered delivery of data packets.
3. Connection State: The OS maintains the state of each connection, tracking information such as the source IP address, destination IP address, port numbers, and the current connection status (e.g., established, closed).
4. Connection Management: The OS handles the low-level aspects of connection management, such as establishing and terminating connections, handling network events, and managing buffers for data transmission.
5. Concurrent Connections: Redis supports multiple concurrent connections from different clients. The OS manages and schedules these connections, allowing them to operate simultaneously.
