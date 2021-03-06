# sockpp
Simple C++11 socket library.

This is a fairly low-level C++ wrapper around the Berkley sockets library using `socket`, `acceptor,` and `connector` classes that are familiar concepts from other languages.

Currently the library only supports IPv4, but soon UNIX socks will be supported, and eventually IPv6.

The base `socket` wraps a system integer socket handle, and maintains its lifetime. When the object goes out of scope, it closes the underlying socket handle. Socket objects are generally _moveable_ but not _copyable_. A socket object can be transferred from one scope or thread to another using `std::move()`.

All code in the library lives within the `sockpp` C++ namespace.

## TCP Sockets

TCP applications are usually set up as either servers or clients. The `tcp_acceptor` is used to create a TCP server. It binds an address and listens on a known port to accept incoming connections. When a connection is accepted, a new, streaming `tcp_socket` is created. That new socket can be handled directly or moved to a thread (or thread pool) for processing.

Conversely, to create a TCP client, a `tcp_connector` object is created and connected to a server at a known address (host and socket). When connected, the socket is a streaming one which can be used to read and write, directly.

### TCP server: `tcp_acceptor`

The `tcp_acceptor` is used to set up a server and listen for incoming connections.

	int16_t port = 12345;
	sockpp::tcp_acceptor acc(port);

	if (!acc)
		report_error(strerror(acc.last_error()));

	// Accept a new client connection
	sockpp::tcp_socket sock = acc.accept();

The acceptor normally sits in a loop accepting new connections, and passes them off to another process, thread, or thread pool to interact with the client. In standard C++, this could look like:

	while (true) {
		// Accept a new client connection
		sockpp::tcp_socket sock = acc.accept();
	
		if (!sock) {
			cerr << "Error accepting incoming connection: " 
				<< ::strerror(acc.last_error()) << endl;
		}
		else {
			// Create a thread and transfer the new stream to it.
			thread thr(run_echo, std::move(sock));
			thr.detach();
		}
	}

The hazards of a thread-pre-connection design is well documented, but the same technique can be used to pass the socket into a thread pool, if one is available.

### TCP Client: `tcp_connector`

The TCP client is somewhat simpler in that a `tcp_connector` object is created and connected, then can be used to read and write data directly.

	sockpp::tcp_connector conn;
	int16_t port = 12345;

	if (!conn.connect(sockpp::inet_address("localhost", port)))
		report_error(strerror(acc.last_error()));

	conn.write_n("Hello", 5);
	
	char buf[5];
	int n = conn.read(buf, 5);
	

	