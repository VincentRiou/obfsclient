## obfsclient - A C++11 obfs2/3 Tor managed pluggable transport client
#### Yawning Angel (yawning at schwanenlied dot me)

> Compile this if you are a obfuscated strong proxy
> who don't need no Python

### What?

This is a C++11 client implementation of the following protocols:

 * [obfs2](https://gitweb.torproject.org/pluggable-transports/obfsproxy.git/blob/HEAD:/doc/obfs2/obfs2-protocol-spec.txt) - The Twobfuscator
   * The shared secret mode is not used in the wild and is unsupported.
 * [obfs3](https://gitweb.torproject.org/pluggable-transports/obfsproxy.git/blob/HEAD:/doc/obfs3/obfs3-protocol-spec.txt) - The Threebfuscator
 * [ScrambleSuit](https://github.com/NullHypothesis/scramblesuit/blob/master/doc/scramblesuit-spec.txt) - *EXPERIMENTAL*
   * This requires tor-0.2.5.x or later for Pluggable Transport arguments.
   * The Session Ticket Handshake is not implemented yet.
   * Protocol Polymorphism is not implemented yet.

By design will only function as a ClientTransportPlugin for Tor.  It does use a
reasonably complete implementation of the Pluggable Transport spec so when used
properly, it will function as a drop in replacement for asn's Python
implementation.

### Building

It currently has the following external dependencies:

 * Compile time only
   * Doxygen (For the documentation)
   * [Google C++ Testing Framework](https://code.google.com/p/googletest/)
     * A copy is included under src/gtest, so there should be no reason to
       install this.
   * [easylogging++](https://github.com/easylogging/easyloggingpp)
     * A copy is included under src/ext, so there should be no reason to install
       this.
 * [libevent2](https://www.libevent.org)
 * [OpenSSL](https://www.openssl.org/)
 * [liballium](https://github.com/Yawning/liballium)

Make Targets:

 * all - Build the obfsclient binary
 * check - Build/Run obfsclient_test
 * docs - Build the doxygen documentation

### Usage

In your torrc:

    UseBridges 1
    Bridge obfs2 ip:port fingerprint
    Bridge obfs3 ip:port fingerprint
    Bridge scramblesuit ip:port password=sharedsecret
    ClientTransportPlugin obfs2,obfs3,scramblesuit exec /path/to/the/binary/obfsclient

### Implementation notes

Like the rest of my C++ code, C++ exceptions and RTTI are not used, and it is
expected that the appropriate compiler flags are passed in to disable these
functions.  The obfsclient binary will assert() on fatal errors (out of memory),
because that's realistically the only safe thing to do.

Caveats:

 * My UniformDH implementation is not quite constant time, though the modular
   exponentiation is.  I do not belive that this is a problem since the
   cryptographic components of obfs3 are intended for obfuscation and not
   secrecy.
 * The UniformDH implementation is glacially slow.  I may be spoiled by using
   Curve25519 so much lately.
 * There is no pushback between the incoming/outgoing sockets, so congestion
   feedback from the bottleneck link will not get propagated.  While this is
   trivial to fix, the official client does not appear to do this either.

### TODO (Patches accepted!)

 * A connection timeout.
 * Implement the missing ScrambleSuit features.

### WON'T DO

 * No, I do not care if this compiles out of the box on Windows.
 * No, I do not care that this doesn't compile with ancient compilers.
 * Unmanaged mode might be a nice to have, but all I care about is Tor.
 * Server implementations of all the protocols (Use the Python version).
