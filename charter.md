# WebTransport (WEBTRANS) Charter
---------------------------------------------

### Chairs
  * TBD

### Applications and Real-Time Area Directors
  * Barry Leiba <barryleiba@computer.org>
  * Alexey Melnikov <aamelnikov@fastmail.fm>
  * Adam Roach <adam@nostrum.com>

### Applications and Real-Time Area Advisor
  * Barry Leiba <barryleiba@computer.org>

### Mailing Lists
  * [General Discussion](webtransport@ietf.org)
  * [To Subscribe](https://www.ietf.org/mailman/listinfo/webtransport)
  * [Archive](https://mailarchive.ietf.org/arch/browse/webtransport/)

### Description of Working Group

The WebTransport working group will define new client-server protocols
or protocol extensions in order to support the development of the
WebTransport API <https://quic.rocks/webtransport-api>.

The WebTransport working group will define an application-layer protocol or suite of
application-layer protocols that support a range of simple communication methods.
These must include unreliable messages (that might be limited by
the path MTU), reliable messages, and ordered streams of reliable
messages.  Attention will be paid to the performance of the protocol,
in particular protocol overheads and the potential for head-of-line
blocking; its ability to be deployed and used reliably under different
network conditions; and the ability to integrate protocol use into
the Web security model. The working group will not define new transport
protocols but will instead use existing protocols such as QUIC and TLS/TCP.

The group will pay attention to security issues arising from
the above scenarios so as to ensure against creation of new
modes of attack.

The WebTransport API will create requirements on the protocols defined
by the IETF WebTransport working group.
To assist in the coordination with the owners of the WebTransport API, the group will
initially develop an overview document containing use cases
and requirements in order to clarify the goals of the effort.
Feedback will also be solicited at various points along the way
in order to ensure the best possible match between the protocol
work and the needs of the WebTransport API.

The group will also coordinate with other working groups within
the IETF (e.g. QUIC, HTTPBIS, TLS) as appropriate. In particular, any changes
to these core protocols shall be discussed in their respective working groups.
The WebTransport working group may define extensions to these protocols,
but any working group last call defining such extensions shall also be sent to
the mailing list of the respective working group.

### Goals and Milestones

 * March 2020 - Adopt a WebTransport Overview draft as a WG work item
 * March 2020 - Adopt a draft defining a WebTransport protocol as a WG work item
 * October 2020 - Issue WG last call of the WebTransport Overview document.
 * January 2021 - Issue WG last call on the first WebTransport protocol document
