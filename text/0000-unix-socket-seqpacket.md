- Feature Name: unix_socket_seqpacket
- Start Date: 2018-04-29
- RFC PR:
- Rust Issue:

# Summary
[summary]: #summary

The goal of this RFC is to extend `std::os::unix` with support for SOCK_SEQPACKET unix sockets.

# Motivation
[motivation]: #motivation

libstd already supports stream and datagram unix sockets, but seqpacket is
missing. Adding it would allow writing applications using it without having to
use `unsafe` directly.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

A sequenced packet socket is connection-oriented like a stream socket,
but sending/receiving transfers whole messages and only one message at a time.

Server applications can use `UnixSeqpacketListener::bind()` to create a socket
which clients can connect to. The server can accept connections with
`UnixSeqpacketListener::accept()`, which returns a `UnixSeqpacket` object for
each connected client. This can be used to communicate with the client.

Client applications can connect to the server's socket with
`UnixSeqpacket::connect()`.

`UnixSeqpacket` provides `send()` and `recv()` methods for communication.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

`UnixSeqpacketListener` creates the socket and allows accepting client
connections, similar to `UnixListener`. `UnixSeqpacketListener::bind()` creates
the socket at the given file system path, while `accept()` allows waiting for
clients. Once a client connects, `accept()` returns a `UnixSeqpacket` object
which can be used to communicate with the client over the socket.

Clients use `UnixSeqpacket::connect()` to establish the connection to an
existing server socket, similar to `UnixStream`, because `SOCK_SEQPACKET` is
connection-oriented.

`UnixSeqpacket` provides `send()` and `recv()` to send and receive individual
messages, and does not implement `io::Read`/`io::Write`, because reads/writes
on a `SOCK_SEQPACKET` transfer whole messages in one go, like `UnixDatagram`.

# Drawbacks
[drawbacks]: #drawbacks

This increases the size of a platform-specific part of libstd.

# Rationale and alternatives
[alternatives]: #alternatives

- The alternative to adding support for this to libstd is to recommend using
  plain unsafe libc directly to open the socket, and then possibly use the
  `FromRawFd` constructors of `UnixDatagram`/`UnixListener` and stick to the
  methods that make sense for `SOCK_SEQPACKET`.

- Since `UnixDatagram`/`UnixSeqpacket` aswell as `UnixListener`/`UnixSeqpacketListener`
  differ mainly only in the socket type parameter, new seqpacket-specific
  constructors could be added to the existing structs, instead of adding
  completely new structs. For example, `UnixDatagram::connect_to_seqpacket()`
  and `UnixListener::bind_as_seqpacket()`. However, `UnixDatagram` has
  `send_to()/recv_from()` methods, which do not make sense for `SOCK_SEQPACKET`.
  Adding the new `UnixSeqpacket`/`UnixSeqpacketListener` structs seems to better
  match the existing design for `UnixStream`/`UnixListener`.

# Prior art
[prior-art]: #prior-art

- [RFC for unix-socket](https://github.com/rust-lang/rfcs/blob/master/text/1479-unix-socket.md),
  which mentioned `SOCK_SEQPACKET` support as (basically) "will be added later".
- [open PR in the old unix-socket crate](https://github.com/rust-lang-nursery/unix-socket/pull/25),
  which adds `SOCK_SEQPACKET` support to the unix-socket crate.

# Unresolved questions
[unresolved]: #unresolved-questions
