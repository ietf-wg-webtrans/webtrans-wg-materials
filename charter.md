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
W3C WebTransport API <https://wicg.github.io/web-transport>.

The WebTransport working group will define a protocol or suite of
protocols that support a range of simple communication methods.
These must include unreliable messages (that might be limited by
the path MTU), reliable messages, and ordered streams of reliable
messages.  Attention will be paid to the performance of the protocol,
in particular protocol overheads and the potential for head-of-line
blocking; its ability to be deployed and used reliably under different
network conditions; and the ability to integrate protocol use into
the Web security model.

The group will pay attention to security issues arising from
the above scenarios so as to ensure against creation of new
modes of attack, as well as to ensure that security issues
addressed in the design of Websockets remain addressed
in the new work.

To assist in the coordination with W3C, the group will
initially develop an overview document containing use cases
and requirements in order to clarify the goals of the effort.
Feedback will also be solicited at various points along the way
in order to ensure the best possible match between the protocol
extensions and the needs of the W3C WebTransport API.

The group will also coordinate with other working groups within
the IETF (e.g. QUIC, HTTPBIS) as appropriate. 

### Goals and Milestones

 * March 2020 - Adopt a WebTransport Overview draft as a WG work item
 * March 2020 - Adopt a draft defining a WebTransport protocol as a WG work item
 * October 2020 - Issue WG last call of the WebTransport Overview document.
 * January 2021 - Issue WG last call on the first WebTransport protocol document
