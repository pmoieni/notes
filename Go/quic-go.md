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
- `exportKeyingMaterial()`: no idea what this does, needs further investigation
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
- `createStream(config *SessionConfig)`: creates either a uni or bi-directional stream.
- `receiveStream()`: removes a stream from the queue of incoming streams. either uni or bi-directional.

## stream features
- `sendBytes([]byte)`: add bytes to the stream `send` buffer.
- `receiveBytes()`: read bytes from stream `receive` buffer.
- `abortSend()`: sends an `abortSendSide` event to the peer including an offset that is reliably delivered.
- `abortReceive()`: sends an `abortReceiveSide` event to the peer including and offset that is reliably delivered.

## protocol events for stream
- `sendSideAborted`
- `receiveSideAborted`
- `allDataCommitted`

> 	 For protocols, like HTTP/2, stream data might be passed to another
      component (like a kernel) for transmission.  Once data is passed
      to that component it might not be possible to abort the sending of
      stream data without also aborting the entire connection.  For
      these protocols, data is considered committed once it passes to
      the other component.
		A protocol, like HTTP/3, that uses a more integrated stack might
      be able to retract data further into the process.  For these
      protocols, sending on a stream might be aborted at any time until
      all data has been received and acknowledged by the peer,
      corresponding to the "Data Recvd" state in QUIC; see Section 3.1
      of [QUIC].

## webtransport datagram spec

A total of 8 bytes in the header:
- first 2 bytes is the type: `webTransportUniStreamType(0x54) / webTransportFrameType(0x41)`
- next 6 bytes for the session ID: `w(http3.HTTPStreamer).HTTPStream().StreamID()`



TODO:
- check if tracing ID exists
	- true: check if session ID exists
		- true: add stream to session
		- false: create new session and add stream to it
	- false: create new tracing ID and set in header, then create new session and add stream to session