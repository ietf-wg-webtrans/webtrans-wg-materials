# WebTransport (webtrans) Working Group Meeting - IETF 124

Chairs: Nidhi Jaju, Lucas Pardue

## Time and Date

* Monday November 03, 12:00 - 13:00 EST ([See this time in your local timezone](https://www.timeanddate.com/worldclock/fixedtime.html?msg=WEBTRANS+at+IETF+124&iso=20251103T12&p1=165&ah=1))
* Location: [Centre Ville](https://datatracker.ietf.org/meeting/124/floor-plan?room=centre-ville)
* [IETF Agenda Link](https://datatracker.ietf.org/meeting/124/agenda/?show=webtrans)

## Links

* [Onsite Tool](https://meetings.conf.meetecho.com/onsite124/?group=webtrans&short=webtrans&item=1)
* [Meetecho Room](https://meetings.conf.meetecho.com/ietf124/?group=webtrans&short=webtrans&item=1)
* [Meeting chat](https://zulip.ietf.org/#narrow/stream/webtrans)
* [Notes](https://notes.ietf.org/notes-ietf-124-webtrans)
* [Minutes](https://datatracker.ietf.org/doc/minutes-124-webtrans/)


## Meeting Notes

Note Takers: Ankshit Jain, Hani Damlaj

1. **Preliminaries**, _Chairs_

   [Note Well](https://www.ietf.org/about/note-well/), [Code of Conduct](https://www.rfc-editor.org/rfc/rfc7154.html), Note Takers, Agenda Bashing, Draft status

- Thank you, David Schinazi! 
   
2. **W3C WebTransport Update**, _Jan-Ivar Bruaroey_

   https://w3c.github.io/webtransport/

- JIB: 90% complete for candidate recommendation
- Do we want to allow web developers to add headers of the CONNECT request? Do we restrict it to the forbidden header names in fetch? Do we allow Authentication headers? If so, what type?
    - WL: If we tie in with fetch, will the headers arrive in time for the server to use to authenticate the establishment?
    - MT: follow existing headers rules for fetch, will need to include cookies
        - These questions are better answered in W3C
    - VV: Agreed, this is for W3C/WHATWG, where we've already had conversations about preflight, etc.
    - MM: Great to allow authentication headers so that everyone doesn't have to implement auth like it was done in WS

3. **Hackathon Interop Results**, _Ankshit Jain, Joanna Jo, Hani Damlaj, Max Inden_
- Safari: 
    - Prototype with latest H2 & H3 drafts 
    - Meta's devious baton server: interop successful with flow control enabled.
        - Session pooling and RESET_STREAM_AT were not tested. 
        - Plans to do WebTransport over H2 interop testing later this week. 
    - Cloudflare's MoQ server: Session established, data exchange working, video streaming has some issues.
- Firefox: 
    - Currently on WebTransport over H3 draft 4, work in progress for draft 14. 
        - Work in progress on flow control and RESET_STREAM_AT
        - No plans to implement session pooling any time soon
    - Meta's Devious baton server: Session established, server opened stream
    - Cloudflare's MoQ server: Video stream working



4. **Working Group Documents**, _Eric Kinnear_
   
   - [Overview](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-overview)
       - Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-overview/pulls
       - Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-overview/issues
       - [Issue #41](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-overview/issues/41): Receive-only unidirectional streams
           - General consensus in the room about receive only unidirectional streams can be solved by a composition of existing operations in wt (e.g. bidi stream, close send side)
   - [HTTP/3](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3)
       - Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/pulls
       - Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues
       - Summary
           - WT_MAX_SESSIONS
               - enforcing monotonically increasing wt_max_streams & wt_max_data (since ordered on the control stream)
       - [Issue #222](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues/222): Additional DoS Considerations
           - H3 considerations apply
           - LP: Yes, I have the action item to add text for this soon. Maybe server vendors are doing certain things that browsers may find surprising.
           - AF: We should figure out recommended wt setting values (e.g. initial wt uni/bidi streams) issues as they might end up showing up in 100% of implementations
           - LP: Same applies to H2 document as well
           - VV: Set some minimum values web applications can expect. MoQ - application suffers if initial window is small
           - EK: Should we define initial values / application profiles?
           - AF: Limits protect receiver memory. Maybe JS app can choose the limits, and JS can handle limit violation?
           - LP: Any suggested number now will probably be a problem later. 
           - AF to submit an issue
       - [Issue #224](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues/224): WT_MAX_SESSIONS
           - Should the peer just try and fail, or do we need a hint to the peer about the limits?
           - CH: This does not affect people who don't do flow control
           - MT: We should remove the setting. What does client do when server is out of capacity for sessions? Recommends new status code for this purpose. We don't do such limiting for CONNECT / WebSockets. 
           - EK: Just to confirm what you're saying, it's okay to just try and not have a hint ahead of time that it's not gonna work. When it doesn't work, we should have a way to indicate that we've reached a limit but we're interested in getting more. It costs clients a round trip to find out this hidden limit
           - MT: WebTransport session establishment is not performance-critical for unhappy paths. 
           - AF: Just tell the peer what the limit is. Burning a RTT and new status code is a worse solution
           - CH: ok to just try attempting a wt session, but need a way to communicate that we're rejecting the request because peer has reached said "limit"
           - LP: Don't like defining a new status code. We have 429 (Too Many Requests), which is maybe more about rate limiting, but there's work going on in the HTTP API WG related to problem types on that response, so maybe we could use something like that instead of a new status code.
           - MB: If limit is 0, client is doomed. If limit > 1, client doesn't need to find out this limit initially. Use the rate-limit API to solve the synchronization problem. 
           - VV: First problem, is that if we remove this parameter right now, the protocol will just break. And it cannot be tied with upgrade token because the upgrade token is request level but the changes we're making are connection level. Finds tri-state flow control negotiation useful.
           - CB: Nothing wrong with having extra data, but it can only be advisory, the server can always change its mind and not tell you, so you will always have to deal with it.
           - EK: Today, you're probably not just supposed to try again, if you're struggling, having all the clients you're trying to tell to go away just come back more aggressively isn't a win.
           - MT: Tri-state thing is an abomination because 0 means one thing and 1 means a different thing and 2 means a different thing but 3 means the same as 2. Someone telling you to use WT for a given URI is a pretty good signal. Rate-limit solution was appealing. 
           - VV: It's not a good signal because you might have an existing connection and if you try to open WT and fail, the server will close all pending HTTP requests with a protocol error.
           - MT: Why?
           - EK: Because you sent a format that the server cannot parse.
           - MT: Extended CONNECT doesn't do that, you can ask for protocols that aren't supported.
           - VV: But the different stream semantics that we're using for WT do, since you can open data streams that show up before the connect and then you got a preamble on a stream that you don't know what to do with.
           - MT: Those streams are going to get killed.
           - EK: Same as if they reject your CONNECT.
           - MT: But they don't look like HTTP streams and the server chokes.
           - VV: WebTransport is exclusively opt in.
           - EK: Maybe we bring back SETTINGS_ENABLE_WT.
           - MT: That's not bad.
           - EK: We go back to a boolean that is just enabled. 
           - MT: Alternative is to do away with the performance optimization and not have a number at all.
           - AF: Rate-limit is fine. I can live with a boolean and a header, that seems totally reasonable to me. 
           - EK: This is also nice because every implementer has a question about why the client sends MAX_SESSIONS when that makes no sense for a client.
           - EK: Server sends ENABLE_WT. If you allow pooling, you hint that. Server uses rate limit headers to send hints.
           - CH: Flow control on top of flow control is a real mess. If we can avoid it, we should. On server side, translate the QUIC settings to another layer. Just defer to QUIC is a good answer here.
           - EK: We avoided nested layers like that where possible (ex: STREAM_DATA)
           - CH: Rate limit works.
           - EK: Don't accept incoming connect request for more than one session unless you sent the flow control settings. Until you send flow control settings, you must only ever accept one session at a time. You should use the rate limit header to signal that you've used your one of one.
       - [Issue #223](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues/223): Stream WT over H3
           - Do we keep separate upgrade tokens / negotiate?
           - Do we allow using WT over a single H3 stream, or keep the requirement to "explode" them out into individual HTTP/3 streams wherever possible? Maybe easier for intermediaries to translate when on a single stream.
           - AF: As a client, I can say "I want to do the stream WT"? 
           - EK: A client can use the upgrade token for webtransport or can say webtransport-h3. Now that you can ask for either, you can ask for either.
           - AF: Is this an API question that needs to be considered by W3C? We have this capability today, with a different name, we'd like to migrate that system here, and so we want this capability. Are there requirements about how this traverses intermediaries?
           - VV: Wants to do this in production. Downstream stacks support streams but don't support webtransport works with this approach. 
           - EK: Okay, we'll put up a PR that removes the restriction, Alan can you help us spell the other half of what you were saying. May need W3C API changes to request which WT upgrade token.
       - [Issue #218](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http3/issues/218): Upgrade token
           - VV: My personal opinion is that we should do the other way because we use WT over H3 right now.
           - EK: For me, we did it first, so I think of it as primary, but from a layering perspective there's one that works everywhere and there's one that's a more specific mapping.
    - [HTTP/2](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http2)
       - Open PRs: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/pulls
       - Open Issues: https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/issues
        - [Issue #143](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/issues/143): Remove SETTINGS_WT_MAX_SESSIONS
            - Doing same for H2 as H2
        - [Issue #155](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/issues/155): Settings parity with QUIC
            - General consensus that we need the distinction here.
        - [Issue #156](https://github.com/ietf-wg-webtrans/draft-ietf-webtrans-http2/issues/156): Handling settings updates
            - You have to track the latest acknowledged values in settings. Should we restrict reducing the values?
            - MT: Current values of SETTINGS are known at the time of the request. Copy the existing values that are valid when it comes in. Use the existing HTTP/2 model for this. The places where we cannot change settings are weird.

   
5. **Wrap up and Summary**, _Chairs & ADs_ 

- LP: Seeing more implementation, which is great. We have pen holders assigned for each open issue.
- LP: Close all pending issues before next meeting, so we can go to WGLC.

- MT: Remind me how we resolved the H3 MAX_SESSIONS setting discussion? What was the answer to that, do we have a summary?
- EK: Put up a PR that removes the setting, changes to server sends WT_ENABLED, server can send hints using rate limit headers, and we have an open question about if we will have a new way to signal establishment failures due to rate limiting.
- MT: It's more important that we resolve it than that we resolve in it some particular way, I'd rather see this go to WGLC in the next few weeks.
