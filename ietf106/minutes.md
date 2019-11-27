# IETF 106 WEBTRANS BoF Minutes

* Singapore
* Wednesday, November 20, 2019
* 13:30 - 15:00
* Room: Collyer

# Preliminaries

* Bluesheet Attendance: 129
* This is a non WG forming BoF
* Minute Taker: Cullen Jennings
* Jabber Scribe: Winston Felix Handte
* Please Join the mailing list: webtransport@ietf.org
* Jabber log:
  * https://www.ietf.org/jabber/logs/webtrans/2019-11-20.html
* MeetEcho Session Recording:
  * http://www.meetecho.com/ietf106/recordings#WEBTRANS
* YouTube link for the Session Recording:
  * https://www.youtube.com/watch?v=o5cJEuO2-vk


# Overview and Requirements (Victor Vasiliev)

https://tools.ietf.org/html/draft-vvv-webtransport-overview

Victor Vasiliev: The problem we are trying to solve is bidirectional communication on the web.  Messages can flow at any time, both ways. We are talking about interactive sessions.  Websockets allowed sending messages in both directions.  RTCDataChannel is SCTP-based, allows partial or unreliable and unordered messages, but is designed for peer-to-peer.  So [for client/server] there is a gap for Reliable but unordered and Unreliable and unordered.  We intend to address that gap with WebTransport.

Target applications - anything that wants one of the following:
- “WebSockets for UDP”
- “WebSockets without head-of-line blocking”

Interest in the following domains:
- Machine learning
- Web games
- Live streaming
- Cloud gaming
- Remote desktop
- Web chat

There are four transports, across two axes.  One axis is dedicated versus pooled.  The other is QUIC versus TCP.  For QUIC we have QuicTransport (Dedicated) and Http3Transport (Pooled). For TCP-based (fallback) we have FallbackTransport (Dedicated) and Http2Transport (Pooled).FallbackTransport is based on WebSockets. Http2Transport is the natural fallback for Http3Transport. QuicTransport has no HTTP/3 dependency, so it is lower latency.


Cullen Jennings:  Is there a requirement for MTU discovery?

Victor Vasiliev: I need to clarify that. In the API we are required to give the MTU.

Tommy Pauly: TCP is not equivalent to QUIC because you won’t get the multi-streaming. My clarifying question: You say it is based on WebSocket.  In what ways is it more than WebSocket?

Victor Vasiliev:  It has more than one stream.

Jonathan Lennox: some of the use cases have requirements for low latency congestion control, which is different from the congestion control required for bulk transfer in QUIC.

Victor Vasiliev: the server you can control this. On client it is more interesting topic. There is an open issue on WICG Github.


Eric Rescorla: Is it required that we standardize all 4 protocols on slide 14?

Victor Vasiliev: It is not required to standardize FallbackTransport. Right half (Http3Transport and Http2Transport) requires standardization because we extend IETF protocols.  The top left one (QuicTransport) requires code point assignment and would be implemented by clients and servers so we need something interoperable.

# Relevant Drafts

## An Unreliable Datagram Extension to QUIC (Eric Kinnear)

https://tools.ietf.org/html/draft-pauly-quic-datagram
https://tools.ietf.org/html/draft-schinazi-quic-h3-datagram

If you were in QUIC this morning, you have seen all of these slides.
We have lots of applications that want reliable control streams and unreliable flows: media streaming, gaming, VPN-style tunneling, and more.

DATAGRAM frames (0x30 and 0x31). Can be done with and without a length.

There is no more flow-id in draft-pauly-quic-datagram (moved to draft-schinazi-quic-h3-datagram).
Supported by multiple implementations, nice interop during the hackathon.

David Schinazi: These drafts will live in the QUIC WG.


Tommy Pauly: For the use of WebTransport, we will have a flow-id Http3Transport but not in QuicTransport.

Bernard Aboba: Victor's presentations will talk about this.

Gorry Fairhurst: Will we multiplex various types of traffic together?

Eric Kinnear: Yes, we should talk.


## HTTP/2 as a Transport for Arbitrary Bytestreams (Eric Kinnear)

https://tools.ietf.org/html/draft-kinnear-httpbis-http2-transport

This is about how we do multiplexed bytestreams over HTTP/2 that can be unidirectional or bidirectional.  Applications include remote IPC or any time where you need multiplexed streams of data. HTTP/2 gives you multiplexed streams, flow control, etc.  We basically use CONNECT to connect through a remote endpoint to somebody else. Can also enable tunneling of UDP, with additional framing.

We’re looking to multiplex several things over a single connection. We want to be able to do this bidirectionally, which means we want to traverse intermediaries in both directions. Would be nice to extend this to support unreliable delivery and datagrams.


Roberto Peon, Facebook: I am curious why we are thinking it is OK to not say what protocol it is we are tunneling? That is useful for load balancers and such to know what.

Eric Kinnear: Makes sense to do something like that


## An HTTP/2 Extension for Bidirectional Message Communication (Guowu Xie)

https://tools.ietf.org/html/draft-xie-bidirectional-messaging

Guowu Xie: Messaging requires bidirectional communications. Clients send messages to the server and get a response, but servers may also want to send a message to clients and get a response. Two major groups of solutions:  stream tunneling and server push.  Neither is good enough for the messaging use case.

In stream tunneling the client establishes a tunnel to the server over a single stream (long polling, websocket, draft-kinnear-httpbis-http2-transport).  HTTP/2 stream is treated like a socket, so client and server have to speak some application protocol over the tunnel. Limitations:  multiple layers of framing, web developers forced to design their own application protocols, reintroduces HoL blocking in HTTP/3, bypasses header compression, bypasses stream prioritization, GOAWAY is less effective.

What about HTTP/2 server push? The lack of acknowledgement makes it unsuitable for our use case.

The extension proposal uses routing streams (RStream) and Extended Streams (XStream).  Xstreams are routed via Routing Streams, so Xstreams do not have to carry headers for routing purposes.

Comparison with WebTransport-over-h3.

Similarities:
- Routing streams and WebTransport Sessions enable routing between the server and client through intermediaries, as well as grouping of dependent streams.
- Xtreams and WebTransport_stream can be created by either peer and allow routing (via routing streams or the Session-Id).

Differences:
- HTTP Message vs. a stream of bytes (WebTransport-over-h3). HTTP Message = structured meta-data (headers) + data (body).  This makes it a better abstraction and a richer building block.

Roberto Peon: To make sure we are on the same page - we are saying that we have a bunch of HTTP messages and we want to make sure that they are all routed to the final destination, correct?

Guowu Xie: Yes

Chris Lemmons (JabberRoom): Do I correctly understand that every proxy between the client and server must support these extended streams for them to operate?

Guowu Xie: Yes


## WebTransport over QUIC and H3 (Victor Vasiliev)

https://tools.ietf.org/html/draft-vvv-webtransport-quic
https://tools.ietf.org/html/draft-vvv-webtransport-http3

Victor Vasiliev: QuicTransport is an application protocol on top of QUIC.  Unlike Websockets, we will extend it as minimally as possible.  Features required to meet WebTransport requirements include:
ALPN value (“wq”)
Client indication (stream with origin)
URI scheme (since this is not HTTP). This includes the host name (sent as SNI), the port and the path (sent in client indication).

HTTP/3 transport gives you a collection of streams in both directions or datagrams within an HTTP/3 connection. Uses extended CONNECT mechanism to establish a session. If the server accepts the session, it returns a new session ID, used to associate all further streams and datagrams with the header.

Lucas Pardue: I was reading the specification for HTTP/3.  You require the extended CONNECT method. In HTTP/2 it is enabled via an H2 setting, but in H3 it is done via a transport parameter?

Victor Vasiliev:  This is one of the details to sort out.


Michael Toonim (Jabber Room): Instead of quic-transport://, shouldn't it just be quic://?

Chris Lemmons (Jabber Room): But, no, I think. quick-transport adds a little to quic.


Tommy Pauly: Lets go back to QuicTransport. Can you clarify the interpretation of path?

Victor Vasiliev: path is whatever the app wants it to be. Talking to users, better to have some path than no path. You could use ALPN to multiplex, but we are using ALPN for security purposes.  Goal is to to be able to multiplex multiple apps on server and we don't have ALPN at this level.

Tommy Pauly: suggest clarifying this usage of it as it is more of a protocol than which resource to use.

Roberto Peon : Lack of multiplexing in websockets was really bad, had lots of limitations due to security and DoS. Have we forgotten this pain? Do we have a solution to this here?  The URI makes me afraid because of DOS concerns. On H2 and H3 we have a smaller attack surface. What threat models are we considering with the exposure of this surface area?

Seph Gentle: What is the status of talking to Web folks about browser APIs.

Victor Vasiliev: There is a draft that is being incubated in W3C WICG.  We have been working on QuicTransport and have a basic implementation. For Http3Transport we are not as far along.


Mike Bishop: In websockets, you say, “I want this resource” and then “I want a web socket on it”. This is backwards in that you say you do websockets then say what resources to do. Not sure how that impacts the security surface. You lose the ability of the server to look at the whole request before making decisions to disclose QUIC transports.

Victor Vasiliev: Lots of things tried not to copy from websockets.


Tommy Pauly: That inversion of semantics may cause issues when falling back to websockets.


Mike Bishop: Clarifying question about how the server accepts the origin.

Victor Vasiliev: There is no explicit OK, because it would add a roundtrip to the handshake. To deny, the server closes the connection.


Ian Swett: Why is the origin sent over a stream instead of QUIC transport parameters?

Victor Vasiliev: Was the original design, but the main problem is that client transport parameters are not encrypted (and we want privacy).


# General Q & A

Eric Rescorla: What is the rationale for QUIC transport?  One thing is less clunky than two things.

David Schinazi: Feedback was developers wanted something very light weight - particularly for game developers (client/server).

Victor Vasiliev: when multiplexing information there are privacy concerns with H3. For example, you can’t ask “give me stats” because it would reveal information for other connections.  Not an issue for QuicTransport.

Eric Rescorla: this sounds like an implementation matter more than a protocol matter.


Martin Thomson: A problem is how these things are identified. Need to understand how this fits into other things like Fetch and CSP.  This doesn't prevent us starting work but we should chat with Web folks. (Martin will file an issue).

David Schinazi: getting to the bottom of this is a W3C question. What we need to get right is the interface between IETF and W3C. The current interface is a URI - may be good enough but if it is not, then we need to get into that but probably at W3C.

Martin Thomson: Some of the security related decisions need to be made at IETF not W3C. And others need to be coordinated with WhatWG.


Roberto Peon, Facebook: Comment: don't take these questions as lack of enthusiasm, I'm asking them because I want this to succeed. Websockets was really, really painful because the group of people that asked for requirements did not understand the requirements. Simple to write and simple to operate are two different things. Websockets was not simple to operate. Server restarts, redirecting to other places. Protocols that deal with multiplexing are the things that have dealt with this. Those practical considerations should dominate.

Brian Trammell: Same comment, questions show enthusiasm. What is state of interfaces in WhatWG and W3C?

EKR:  It has no W3C status.


For posterity, links to the documents are pasted below:
- What WG:
  - Streams (basis for both WebSocketStream and WebTransport):
    - https://streams.spec.whatwg.org/
  - WebSocketStream API status: likely to be handled in WhatWG.
    - https://docs.google.com/document/d/1La1ehXw76HP6n1uUeks-WJGFgAnpX2tCjKts7QFJ57Y
- W3C:
  - WebTransport is being incubated in WICG.
    - https://wicg.github.io/web-transport
    - https://github.com/WICG/web-transport
    - https://discourse.wicg.io/t/webtransport-proposal/3508/7


Brian Trammell: This means we need tighter look than we have done before with W3C.

Magnus Westerlund (Jabber Room): WebRTC and RTCWeb had that rather tight interaction for something that had a substantially bigger scope than this.

Bernard Aboba: Protocols last longer than APIs.  Feedback would go mainly from IETF to W3C, and many of the questions you are asking would be answered here, rather than creating an endless loop and lots of liaisons.

Mark Nottingham (as W3C liaison): There is no official W3C status yet, but the W3C is pretty keen about this, so I would not take the lack of status as a lack of interest.  We can make the liaison work.

David Schinazi: We've started the feedback loop between W3C and IETF, this BoF is the product of that loop.


Brian Trammell : It looks like a stack specification is bubbling up into the spec. (See URI). This looks like recurring patterns and the IETF should get it right once.

Lucas Pardue: This is a quite broad extension of H2/H3. We should clarify how this interacts with Alt-Svc and the fallback.

Eric Kinnear, Apple: We are excited to work on this. Lots of overlap on what people's needs are. We've started work on seeing how we can combine efforts.

Cullen Jennings: A serious mistake on how we set up liaisons was that we did not come to agreement on whether the W3C can remove security features that IETF considered to be mandatory.  I would oppose us moving forward without us having that very clear before we get started.

Bernard Aboba: Do you have specific security concerns?

Cullen Jennings: Nothing specific, we just need to be clear on the W3C/IETF interaction.

Eric Rescorla: On the topic of W3C/IETF joint work, I am not sure that either of the last two API/protocol development efforts worked out particularly well.  WebRTC is like, 27 years old. We'll need to be careful. Worried about folks losing interest at end of project.


# Pointer to Charter Discussion

Bernard Aboba:  This is a pointer to the charter discussion, since this is not a WG-forming BOF.  There has been discussion on the mailing list (webtransport@ietf.org) and the charter is up on github (https://github.com/DavidSchinazi/webtrans-wg-materials)
So if you have opinions, you can post them to the mailing list or file an issue on Github. I will show you where we are, so you can express your opinions on the mailing list or on Github. (Shows latest charter proposal posted to the mailing list).

Mirja Kühlewind: What are the implications for QUIC here?

David Schinazi: This just uses QUIC, it has no new requirements on HTTP or QUIC WG (except we need QUIC to have the DATAGRAM extension).

Mark Nottingham: You are using HTTP.  I noticed a new pseudo-header, for example. This is stuff that needs review in HTTP WG.

David Schinazi: Absolutely, charter states than standard actions will be sent to HTTP mailing list.

Ray ?: Any chance we could get you to change the name? It’s neither a transport nor does it have anything to do with the web.

Ted Hardie (from the Jabber Room): The Holy Roman Empire was not Holy, not Roman, and not an Empire:  Discuss amongst yourselves.(https://en.wikipedia.org/wiki/Coffee_Talk)

Eric Rescorla: Agree, this is not a good name.

David Schinazi:  Bikeshedding on the name is not a clarifying question.

# Wrap up and Summary

David Schinazi: This is not a WG forming BOF. We’re going to have hums to gauge interest and determine how much change is required.

Question #1: Is the WebTransport problem statement clear, well-scoped, solvable, and useful to solve?

Room hummed more for Yes than No.

Jabber: two yesses.

David Schinazi:  If someone hummed no, can you say why?

Roberto Peon: I have a minor concern on whether scope should increase.

Question #2: Are the WebTransport deliverables (WebTransport overview, QuicTransport, Http3Transport, FallbackTransport) well-defined and well-understood?

Ted Hardie: I think some of these are well-defined but not others.

David Schinazi: Can you say which ones are well-defined and which are not?

Ted Hardie: The relation between QuicTransport, Http3Transport and fallback needs some fleshing out. Suitability of fallback may be variable. For example, what the security properties are when falling back may be different when not falling back. And we can probably get to well defined deliverables but I am concerned that we are not there yet.

Eric Rescorla: In what sense is this not a WG forming BoF?

Bernard Aboba (as chair): We're trying to understand what parts of a potential charter need more work.

Barry Leiba (as responsible AD): I don't think we went through charter bashing significantly. The question here is whether the technology is well defined and well understood.

Eric Rescorla: Is one possible outcome that the next thing we see is charter and proposal to form a WG without BoF.

Barry Leiba: That'll depend on the discussion on mailing list. We might be able to charter without a BoF, but that'll be an outcome of the mailing list, not an outcome of this BoF.

Eric Rescorla: The email mentioned a non WG-forming BoF and these hums sounds like a WG-forming BoF. Please me clearer so people know to attend sessions or not. Let's talk offline.

Mirja Kühlewind: My understanding from the mic line is that the proposal is not entirely well defined.

Bernard Aboba (as chair): Can you be more specific, what items are not well defined?

Mirja Kühlewind: At a high level today this is fine but some details in the drafts need far more discussion.

Victor Vasiliev: How is question #2 different from question #1?

Bernard Aboba (as chair): We're trying to find what items need to be worked on more.

David Schinazi (as chair): From the mic line feedback, question #2 is not well defined. We're not going to take a hum on question #2 today.

Tommy Pauly: These deliverables are not the right granularity. These are one way to solve the problem but the charter should talk more about the problem statement than a given solution.

Jonathan Lenox: It's not clear that there should be 3 transports. That decision should be made by the WG instead of being in the charter.

Chris Lemmons (Jabber Room): Then let's hum about what needs more work. Instead of humming about the whole list for adoption.

David Schinazi: we're not going to have those hums today.

Martin Thompson: I hummed against the first hum. We need to clarify the problem statement. We need to clarify the relationships between organizations. The way way theses things need to be identified and managed. Drafts are unclear in this regard.

Brian Trammel: +1 to what Martin said. We should emphasise a bit more the overview and I'm not sure these deliverables are all needed. Relation for security is meh. This is one proposal. There are other ways to do this, we should get the overview out first.

Mark Notingham: +1 to what Brian said. I'm concerned about how complicated this is, there are many moving parts. This is too complicated and it could be defined as an evolution from websockets.

David Schinazi: Taking a show of hands on people willing to review docs.
About two dozen hands were raised.

David Schinazi: To wrap up:
- the general problem statement is pretty well understood
- there is interest in the community to work on this
- we need to put more energy into the charter as the current draft charter is too specific and the deliverables might not be right ones

Mirja Kühlewind (as transport AD): I was involved in making this a non-WG-forming BoF and encourage you to put more energy into the charter and individual proposals

Barry Leiba (as responsible AD) show of hands for people in the webtransport mailing list or planning to join it soon [quite a few hands raised]. Please focus on refining #1 on the mailing list.
