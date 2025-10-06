	The central entrypoint into quic-go is the quic.Transport. It used both when running a QUIC server and when dialing QUIC connections.

Unlike TCP which uses the tuple of `ip_address` and `port`, Quic uses Connection IDs

## Transport protocol requirements

- rate limiting
- multiple sessions between same client and server
- validate origin
- sub-protocol support
- must provide datagram support
## session requirements
- may allow 0RTT

## session features
- `createSession(origin)`: creates a session given the URI, `origin` is optional if NOT browser
- `terminateSession() (error uint32, reason [1024]byte)`: no more application data will be exchanged, error and reason are optional with `0` and `""` default values respectively
- `drainSession()`: drain the session from any traffic and indicate to the peer for a graceful termination 
### session event types
- `sessionTerminated`: indicating that the session has been terminated, may provide error code and message if terminated by the peers
- `sessionDrained`: indicating a possible drain request from peers. new streams can be opened but is discouraged. 

> note: server selects the protocol based on the preference order

## Datagrams
slice of bytes with a maximum length of path MTU (usually 1200, sometimes 1500) that is NOT expected to be transmitted reliably (unlike streams)

> note: webtransport itself is not expected to re-transmit datagrams but the underlying protocol may implement safety measure like TCP

- not flow-controlled
- peers should be provided with max datagram size during 0RTT and 0.5RTT using PMTUD
- all of the outgoing and incoming datagrams
   are placed into a size-bound queue (similar to a network interface
   card queue) 
   > note: figure out what NIC queues are

## session protocol features
- `sendDatagram([]byte)`: Enqueues a datagram to be sent to the peer. potentially resulting in the datagram being dropped if the queue is full.
- `receiveDatagram([]byte)`: Dequeues an incoming datagram, if one is available.
- `getMTUD() uint`: Returns the largest size of the datagram that a WebTransport session is expected to be able to send.
