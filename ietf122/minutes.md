# WebTransport (webtrans) Working Group Agenda - IETF 122

Chairs: David Schinazi, Nidhi Jaju, Lucas Pardue
Note Takers: Martin Thomson, Ankshit Jain

## Thursday, 20 March 2025

### **Preliminaries** -- _Chairs_ (15 minutes)

   [Note Well](https://www.ietf.org/about/note-well/), [Code of Conduct](https://www.rfc-editor.org/rfc/rfc7154.html), Note Takers, Agenda Bashing, Draft status

In memoriam: Bernard Aboba
David recounts a story about RFC 5218.  "All the great protocols came from outside the IETF."

### **[W3C WebTransport](https://w3c.github.io/webtransport/) Update** -- _Will Law_ (15 minutes)

Will reads the slides.
Many people in the W3C group are in the room.
Almost done!
Candidate recommendation in 1-2 months. Hope to be completely done before the W3C meets in November.

Updates 
- redirection as network error
- protocol constructor 
- datagrams.createWritable() replaces datagrams.writable
- priorities with sendOrder, sendGroup - shared fairness between send groups, with sendOrder differentiating within sendGroup,
Issues: 
- #637: CORS for WebTransport: Is CORS required if ad-hoc headers are allowed? 
- #623: Consider removing serverCertificateHashes: 
- #264: Expose CONNECT response headers: Should allow adding headers for CONNECT request - some support online
    - EK: "Why not?", people might send headers after the connection is established, so we should allow sending headers at the CONNECT request.
    - VV: Previous opinion: send in payload, now changed to headers are received before application data, so this is good. Allows pren
    - MT: if we allow headers, we should also allow cookies, which open the CORS question, we currently engage with Fetch at a lower level and this would entail a higher-level engagement - this is not really a question that the IETF should be answering; the CORS question is very relevant here.
    - VV: \<missed what Victor said, check audio\>
    - chairs: question is really for the W3C to resolve

### **[WebTransport over HTTP/2](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http2) and [HTTP/3](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3)** -- _Victor Vasiliev_, _Eric Kinnear_ (40 minutes)

Open Issues: 
- #172: Subprotocol negotiation: should it be mandatory? Victor has shipped some code which makes it NOT mandatory, protocol could be inferred from the URI instead of host+port in QUIC
    - MT: "Victor is right"
    - DS: some confusion over what was in the API (resolution was that this is optional in the API, for the reasons that Victor stated)
    - DS: Consensus in the room to make it optional

- #150: Negotiation of flow control: questions if we skip / don't skip flow control
     - EK: who (browsers, AQM) gets to choose how things are prioritized? 
     - EK: Do we need another mechanism to negotiate flow control: 
         - Option 1: client sends MAX_SESSIONS again, if > 1 on either side FC is enabled, == 1 no flow control, 0 no WebTransport
    - MT: "3 numbers in computer science 0, 1, not 1". "Flow control is not that hard", prefers everyone does flow control, connection is shared with HTTP and WebTransport so it is useful. 
    - VV: i don't expect people to implement if we ship it that way; I can't implement this; it requires that people understand reset offsets; that's not something that is typically exposed through QUIC APIs
    - EK: Recap: VV suggests that if people are required to implement this, they won't implement webtransport at all
    - DS: "Hard to implement in Chrome stack, easy to implement in MT's stack, everyone can live with it being negotiable"
    - AF: Martin's point about HTTP requests on the same connection.  Right now it's about limiting streams, so maybe not so useful
    - KO: Complexity difference is not big from browser side, more difference on proxy side
    - EK: As an intermediary, how to suggest one H3 connection to a back end going down can be handled?  How do you stop the client from filling buffers 
    - KO: don't do that then
    - VV: Don't think FC is a problem,this is about different rates at which the backend can consume content from different sessions; different sessions can consume data at different rates
    - EK: browser can choose to limit what JavaScript is able to do, than server can offer. Server can send large values and not worry about it. 
    - LP: Previously had reservations about flow control, but has worked on a bug recently which \<missed what Lucas said, check audio\>
    - AF: Not implemented flow control, want to do before Madrid, need interop experience with flow control.
    - DS: Land negotiation, implement flow control. If implementation is easy, we remove negotiation, otherwise we keep it negotiable. Requesting thoughts about the solution from the room. 
    - VV: Simplicity of deployment is main point of WebTransport \<missed what Victor said, check audio\> 
    - MT: everyone should implement flow control, otherwise advertise a value UINT64_MAX
    - DS: Interop target draft will be decided soon
    - EK: Not sure if this is the same level of complexity as building your own QUIC stack. flow control and pooling go together. 
    - DS: Land the negotiation, cut a draft, people implement, discuss in Madrid regarding implementation
    - EK: "Are we all on same page that flow control and pooling go together?"
    - DS: If implementations suggest flow control is hard, remove flow control and pooling. Discuss post learnings from implementations. EK will write the PR for negotiation. 
    - BS: Flow control for counting streams is easier than counting bytes. Keeping stream count limits might be worth it. 

- #179: Requirements on final size
    - EK: are implementations comfortable with WT_MAX_DATA? Wait until Madrid/until we have implementation experience? 
    - MT: No, there are no W3C implications. 
    - AF: Asks the chairs to schedule virtual interop as implementations may not happen by Madrid. 
    - KO: If this is complicated, MAX data can be same as QUIC limit

- EK: Will check regarding subprotocol negotiation being optional.
- DS: Pending PR on MT
- DS: Will decide on virtual interop

HTTP/2:

- EK: Assistance requested from chairs regarding interop availability
- AF: Explore if devious baton is good for interop
- AJ: Apple has devious baton ready for interop

- DS: \<missed what David said to Victor regarding key exporters, check audio\>


### As Time Permits: **[Forward and Reverse HTTP/3 over WebTransport](https://datatracker.ietf.org/doc/html/draft-various-httpbis-h3-webtrans/)** -- _Ben Schwartz_ (10 minutes)

Fine interface line between HTTP/3 and QUIC - shares similarities with WebTransport session. 
- BS: Adoption in HTTPBIS/WEBTRANS?

- PS: Should work ...
- AF: Didn't realize stream IDs till mentioned, how important is this for getting it working i nt he 
- Y: Created the draft with the assumption that WT is not providing stream IDs. GOAWAY and DATAGRAMS require stream IDs so they are handled separately. 
- AF: Convinced by the use case of origin?, not sure if this is the best solution. Can implement quickly. 
- EK: Private relay has a series of layered protocols, so not sure about this solution. H3 has interesting layering violations regarding QUIC streams, as it fully consumes QUIC connections. Having dependency on stream IDs could be painful. 
- VV: Can? reuse actual QUIC stream IDs, stream IDs may not match if going through proxy, \<missed what Victor said, check audio\>
- BS: DATAGRAMS with WT is inconvenient without stream ID association. Fixing this on intermediaries may be challenging.
- DS: ?
- MT: Reverse HTTP work should happen in its own WG. Suggests this is a bad option. HTTP on QUIC could be done in reverse - new ALPN, reverse it. URI addressing could be limited. Minimum number of layers should be explored. Don't want to run web servers in browsers. 
- KO: Agrees on the use case, suggests setting up H3 and sending the request in reverse? 
- BS: Has thought about it? Likes the flexibility of HTTP authentication here. Not clear about HTTP authentication over QUIC in reverse. MTLS requires exploring certificates to use. 
- LP: Likes the idea of a standard for such use cases. Maybe this needs its own WG. Requires people from proxy, gateway areas which may not be in WEBTRANS WG.
- DS: Not sure about this solution. Highlights the interest in the problem space from the room, building something on top of WebTransport is not in the charter or WEBTRANS WG. Number of different ways to do reverse HTTP. No call for adoption, but there is interest. Suggests to follow the process used for MASQUE. 

### **Wrap up and Summary** -- _Chairs & ADs_ (10 minutes)

Talked for plan for interop.
Thank you DS!

