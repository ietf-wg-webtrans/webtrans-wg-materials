# WebTransport (webtrans) Working Group Agenda - IETF 123

Chairs: David Schinazi, Nidhi Jaju, Lucas Pardue
Note Takers: Marco Munizaga

## Wednesday, 23 July 2025

Slides: https://datatracker.ietf.org/meeting/123/materials/slides-123-webtrans-webtransport-slides-ietf-123-03

## Preliminaries, Chairs (15 minutes)

## W3C WebTransport Update, Will Law (15 minutes)
https://w3c.github.io/webtransport/

(see slides)

serverCertificateHashes discussion current agreement is to keep it in the spec.

First meeting we have no items we need feedback or discussion on.

Ben Schwartz: Can this API be implemented over the abstract API defined in the overview.
The w3c API lives on top of the abstract layer defined in overview which is instantiated by the h2 or h3 spec.

Can the new commit message be implemented on top?

Eric K: Yes, w3c has done work such that it is built on top of the overview rather than a specific transport. The commit api for example is a no-op in h2 as everything is commited once sent to the TCP buffer.

Ben S: In the overview, It has an event called all data commited, but there's no commit method

Victor V: All data in h2 is commited when written to the TCP buffer

Ben S: We might still need a new method in the overview draft to address the commit method.

## Hackathon Interop Results, Alan Frindell (15 minutes)

_Devious Baton_ protocol to exercise WebTransport features.

Show of hands who has a webtransport implementation? 3 or 4 hands.

David S: Who is interested in MoQ? (3 ish hands)

I invite browser implementors to come to the mic and talk about your plans

Eric K: This is the first time I've seen 2 of 3 browsers that can support the protocol. I start seeing the energy build

Alan F: How soon can we expect feature support?

Eric K: You can see the work happen in the open 

Victor V: It is unlikely I have time to work on anything not related to MoQ. There may be other people at Google or the open source project that may have time to work on other parts.

Alan F: So things like version negotiation, what about flow control?

Victor V: No plans to implement the flow control. If someone is planning on implementing it chrome, that would be fine with me.

Max I: Currently on draft 4. on our roadpmap, no comment on timeline

Marten S: Where do we stand on WGLC on the h3 document? Folks are holding off on work on the protocol until it progresses more

David S: Close to done on open questions and issues on the protocol. Won't send to IESG until its implemented. Current plan is to have folks implement. We think the doc is in good shape to implement. The only changes expected are for clarity or bugs in the spec. We are on a holding pattern waiting for all features or close to all features to be implemented.

## WebTransport over HTTP/2 and HTTP/3, Eric Kinnear (35 minutes)

(see slides)

https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3
- Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/pulls
- Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues

https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http2
- Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/pulls
- Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/issues

https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-overview
- Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-overview/pulls
- Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-overview/issues

We are in the pencils down, implement it moment.

subprotocol negotiation is now in the overview.

If flow control is not enabled, ignore flow control capsules

Open question: Should we require the "MUST implement flow control..." clause when sharing WebTransport with HTTP traffic?

Lucas P: Clarification, when we say all HTTP Traffic, and HTTP connection may not carry only HTTP traffic (e.g. websocket)

Eric K: It should say non-webtransport traffic. 

Victor V: Flow control with WebTransport is not a security issue. Even with two regular HTTP responses, there are no flow control between them. What can happen is you can have a WebTransport use all you flow control and no longer be able to open HTTP requests.

Eric K: (repeating the point) Because you are going to the same origin, we say don't stomp on each other, and if you do that's your problem.

Victor V: yes. The main purpose of embedding flow control is when you have multiple sessions with multiple backends and allow the proxy to not have to buffer. Flow control is not there to prevent the server from shooting itself.

The server should not be there to tell you no.

Alan F: One thing I don't agree with: The HTTP response case with two responses stomping on each other. We have stream flow control for this, so I don't think that can happen.

For the original question, I think we want this.

Nothing stops the HTTP session from using the whole flow control and stomp on WebTransport.

Marten S: Agree with Victor that flow control is not a security issue. Should we allow this with HTTP. Yes, I think we should.

Eric K: Should we allow or require?

Marten S: I would want flow control on every case, but don't want to open that can of worms

David S: Back to the original question, should the spec require it? (Marten shrugs)

Lucas P: Agree not a security consideration. Might call it an availibility considerations. Could link a 10 year bug that affects cloudflare and customers. The bug is that if there's a video loading, it may consume all the flow control and cause everything else to stall/fail.

We should make others aware of this consideration. No opinion if it must be a MUST or not, but at least folks should be aware.

### 214 Explicit negotiation of flow control

Eric K: Do we want this? (Nods in the room)

How?

Victor V: Mildly prefer A. We can be clever but choose to be clearer.


Marten S: First preference is option D, second preference for option B.

Eric K: Do you have a synchronization problem? I as the client have to wait for the server settings. 

Marten S: Yes, but doesn't cost a round trip as server settings are sent in 0.5 RTT. Client can't open WT until it receives the settings. 

Eric K: right so A would also be fine

David S: Anyone object to B?

Eric K: Any issue with the client settings not arriving in time? No

Consensus in the room to go with option B

---

Eric K: We'll move security consideration around flow control to a performance consideration section

---

Structured fields slide

Victor V: This change is not backwards compatible.

Eric K: That's fine since we changed the transport parameter version.

Victor V: Just noting the transition from token to string.

David S: In the structured fields RFC, tokens are only defined for backwards compatibilty, Strings are recommended

---

Drain session slides

Ben S: What's the difference between wrap up and drain?

David S: We don't have wrap up "wrapped up", so we can't use it just yet. We'll keep it separate for now, because wrap up is not ready.

Eric K: Are there things that we would want from the wrap up work?

David S: We won't know until we've done all that work. If we learn something from wrap up, we could update WT and add the wrap up things or use wrap up instead

---

PR #132

Eric K: I thought we agreed on Webtrans-over-h2 was somewhat generic to the bytestream, but we should scope it to just h2

Victor V: For h3 we should just mention it. For h1, the draft is ambigous because by defining the upgrade token we can kind of do this with h1, but by being explicit we can be clear about the security issues around web h1.

This was also motivated by Qmux. Webtransport over h2 is a well defined way of doing this over something that looks like a tcp connection. As to why we're filling in the gaps now, there isn't that much text to write now, and there's motiviation use cases for this (see QMux)

David S: Checking charter, it doesn't explicitly prohibit this, when we adopted the document the scope was specifically webtrans over h2. We could change the document scope, but consider:

- We should coordinate with HTTP wg

Alan F: Want folks to be aware of QMux work in the quic wg. I don't think it makes sense to define how to use this outside of h2 in a doc called wt over h2. But I do have a use case for this.

Kazuho: It's not unreasonable to run webtransport over non h2, at least for testing purposes. I would say extended connect is not a version independent property. We don't have this for all non-h3 http versions. We should instead be explicit on exact HTTP versions or how extended connect works in HTTP.

Lucas P: It seems weird to put these statements in the h2 document. We probably need to sort out the details with Qmux overlap/interaction. But I'd like to have the document serve the protocol implementor.

It reminds me of the binary HTTP work that was done.

David S: speaking as individual. I don't think we should add this work. It would be time consuming and would rather ship this sooner. We can do soemthing like this somewhere else (qmux)

Victor V: I assume you're referencing the non-http case

David S: I'm referring to the part that refers to new protocol elements. An appendix that highlights some issues seems reasonable. A normative section, speaking as chair, would delay the document.

Victor V: I don't think defining this over h1 is much more work.

I agree it makes more sense to move things into a new draft.

Ian S: As an individual, having just the h1 version would be fine for me. Since h1 is always going to be the fallback.
That said, putting this in a h2 document is bizarre.

Eric K: We started with an h2 specific thing, but we've transformed it over time to a arbitrary bytestream. I'm not sure the extra protocol definitions would make sense in this doc.

Nidhi: Sounds like there is no consensus on adding this section, but may be okay as an appendix.

---

QMux

Lucas P: Preview of the QMux work. 
We had a interim meeting in June. The outcome:

Consensus: Appropriate for wg. Would require a charter update. Sent to to the IESG.

In the quic wg, Kazuho will outline some possible solutions to discuss.

Eric K: had we had a different timeline it seems likely that we would have done Webtransport over QMux. But as it stands today, it makes sense to have pencils down moment here with h2.

---

What's left?

Harald A: server certificate verification is unspecified in the current doc, but is referenced in the w3c api. 

Victor V: RFC 8446 is not a webtransport concern. There shouldn't be anything to mention in the webtransport spec.

David S: We should move this issue to the webtrans-overview spec with some text that mentions how verification is done with a reference to RFC 8446.

## Wrap up and Summary, Chairs & ADs (10 minutes)

Interop success results didn't happen exactly as planned.

folks are implementing but folks are not yet ready with the latest draft.

we'll start wglc on these 3 docs. The idea being if there are changes after that, that will be the interop target.

We won't send docs to be published until we have interop results and implementations.
