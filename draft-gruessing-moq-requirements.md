---
title: QUIC Encapsulation for Media over RTP - Requirements and Use Cases
abbrev: MoQ Requirements and Use Cases
docname: draft-gruessing-moq-requirements-latest
category: info
date:

ipr: trust200902
area: applications
workgroup: AVTCORE/MMUSIC Working Groups
keyword: Internet-Draft QUIC RTP
stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, subcompact, comments, inline]

author:
  - ins: J. Gruessing
    name: James Gruessing
    email: james.ietf@gmail.com
  -
    ins: S. Dawkins
    name: Spencer Dawkins
    organization: Tencent America LLC
    country: United States of America
    email: spencerdawkins.ietf@gmail.com

normative:
  RFC9000:
  I-D.draft-ietf-webtrans-overview:
  I-D.draft-ietf-mops-streaming-opcons:
  I-D.draft-ietf-quic-datagram:

informative:
  I-D.draft-dawkins-sdp-rtp-quic-questions:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-kpugin-rush:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-sharabayko-srt:
  I-D.draft-ietf-quic-http:
  RFC3550:

--- abstract

This document describes the uses cases, requirements, and considerations that
should guide the design of the encapsulation of a real-time media transport
protocol as a payload in the QUIC protocol.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

--- middle

# Introduction {#intro}

This document describes the uses cases, requirements, and considerations that
should guide the design of the encapsulation of a real-time media transport
protocol as a payload in the the QUIC protocol {{RFC9000}}.

Protocol developers have been considering the implications of the QUIC protocol ({{RFC9000}}) on media transport for several years, but the initial focus on QUIC in the IETF was to support web applications that used the HTTP/3 protocol {{I-D.draft-ietf-quic-http}}. The completion of the initial versions of the QUIC specifications, and the adoption of {{I-D.draft-ietf-quic-datagram}}, have cleared the way for proposals to use QUIC as a media transport. This document considers a number of proposals for "Media Over QUIC", and analyzes them to understand requirements and considerations.

## Relationship with draft-dawkins-sdp-rtp-quic-questions

Work on this draft and on {{I-D.draft-dawkins-sdp-rtp-quic-questions}} began in isolation from each other, and the authors of both drafts are still working out the scope of each draft, so there is some overlap between the drafts. At this time, it's worth saying that

* This draft is intended to capture requirements for the encapsulation of Media Transport Protocols in the QUIC protocol

* {{I-D.draft-dawkins-sdp-rtp-quic-questions}} is intended to capture questiona that will affect the specification of Session Description Protocol description for these encapsulated Media Transport Protocols

Both specifications are open for issues and pull requests, and since there is a common author, assume that any contribution you make to either draft in GitHub will end up in the right place.

# Terminology {#term}

For the purposes of this document, it is assumed that we are starting with a protocol stack that looks like this:

~~~~~~
    Media Transport Protocol | Media
~~~~~~

and the goal is to provide a protocol stack that looks like this:

~~~~~~
    QUIC | Media Transport Protocol | Media
~~~~~~

# Prior and Existing Specifications {#priorart}

Several existing draft specifications and protocols already exist which base
their implementation around using existing Media Transport Protocols on top of QUIC, or define
their own. With the exception of RUSH ({{kpugin}}), it is unknown if the other specifications
have had any deployments or interop with multiple implementations.

## QRT: QUIC RTP Tunnelling {#hurst}

{{I-D.draft-hurst-quic-rtp-tunnelling}}

QRT encapsulates RTP and RTCP and define the means of using QUIC datagrams
with them, defining a new payload within a datagram frame which distinguishes
packets for a RTP packet flow vs RTCP.

## RTP over QUIC {#englebart}

{{I-D.draft-engelbart-rtp-over-quic}}

This specification also encapsulates RTP and RTCP but unlike QRT which simply
relies on the default QUIC congestion control mechanisms, it defines a set of
requirements around QUIC implementation's congestion controller to permit the
use of separate control algorithms.

## RUSH - Reliable (unreliable) streaming protocol {#kpugin}

{{I-D.draft-kpugin-rush}}

RUSH uses its own frame types on top of QUIC as it pre-dates the datagram
specification; in addition individual media frames are given their own stream
identifiers to remove HoL blocking from processing out-of-order.

It defines its own registry for signalling codec information with room for
future expansion but presently is limited to a subset of popular video and audio
codecs and doesn't include other types (such as subtitles, transcriptions, or
other signalling information) out of bitstream.

## Tunnelling SRT over QUIC {#sharabayko}

{{I-D.draft-sharabayko-srt-over-quic}}

Secure Reliable Transport (SRT) ({{I-D.draft-sharabayko-srt}}) itself is a general purpose transport protocol
primarily for contribution transport use cases and this specification covers the
encapsulation and delivery of SRT on top of QUIC using datagram frame types.
This specification sets some requirements regarding how the two interact and
leaves considerations for congestion control and pacing to prevent conflict
between the two protocols.

## Comparison of Existing Specifications

* Both QRT and the Engelbart draft attempt to use existing payloads of RTP,
  RTCP, and SDP unlike RUSH and SRT, as well as using existing Datagram frames
* RUSH introduces new frame types as its development pre-dates Datagram frames
* All drafts take differing approaches to flow/stream identification and
  management; some address congestion control and others just omit the subject
  and leave it to QUIC to handle
* Both QRT and RUSH specify ALPN identification; the Engelbart and SRT drafts do not.

# Use Cases {#usecases}

## Use Cases From {{I-D.draft-rtpfolks-quic-rtp-over-quic}}

An early draft in the "media over QUIC" space, {{I-D.draft-rtpfolks-quic-rtp-over-quic}}, defined several key use cases. The following sections are taken from that draft, with minimal editing.

### Interactive peer-to-peer applications

Interactive peer-to-peer applications, such as telephony or video conferencing.  Such applications operate in a trapezoid topology using a client-server signalling channel running SIP or WebRTC,  and an associated peer-to-peer media path and/or data channel. Mappings of SIP and WebRTC onto QUIC are possible, but outside the scope of this memo.  It might be desirable to transport the peer-to-peer RTP media path and data channel using QUIC, to leverage QUIC's security, stream demultiplexing, and congestion control features running over a single UDP port. This would simplify media demultiplexing, and potentially obviate the need for the congestion control work being done in the RMCAT working group.  The design of QUIC makes it difficult however, since QUIC does not support peer-to-peer NAT traversal using STUN and ICE (and indeed uses a packet format that conflicts with STUN). These applications require low latency congestion control, and would benefit from unreliable delivery modes.

### Interactive client-server applications

Interactive client-server applications. For example, a "click here to speak to a representative" button on a website that starts an interactive WebRTC call.  Such applications avoid the NAT traversal issues that complicate peer-to-peer use of QUIC, and can benefit from stream demultiplexing and (if appropriate algorithms are provided) congestion control.  They would benefit from unreliable delivery modes to reduce latency.

### Client-server video on demand applications using WebRTC or RTSP

Client-server video on demand applications using WebRTC or RTSP. These benefit from QUIC stream demultiplexing in the same way as interactive client-server applications, but with relaxed latency bounds that make them fit better with existing congestion control algorithms and reliable delivery.

### Live video streaming from a server

Live video streaming from a server can also benefit from stream demultiplexing.  If designed carefully, it should be easier to gateway RTP over QUIC into multicast RTP for scalable delivery than to gateway HTTP adaptive video over QUIC into multicast.

## Suggested Use Cases for "Media Over QUIC" Going Forward

The use cases that are most applicable today given the existing and known future capabilities of QUIC are included in this section.

**Editor Note:** this section is a work in progress, and is based on the opinions of the draft authors. We are happy to be guided by discussion about other use cases.

### Unidirectional live stream contribution

Unidirectional live stream contribution. Two immediate scenarios that
best describe this is firstly users on a streaming platform in a remote scenario
from their phone live streaming an event or going on to an audience in real time
in relatively low bitrates (~1-5Mbit). The second scenario is larger bitrate
contribution feeds in broadcast. This can be an OB feed "back to base" into
playout gallery, or from playout facilities to online distribution platforms.

### Distribution from platform to audience

Distribution from platform to audience. Whilst use of WebRTC or RTSP today
for On-Demand media streaming is not typical with adaptive streaming like HLS
and DASH being predominantly used as WebRTC is more applicable in latency
sensitive contexts such as live sporting events. Instead use cases where there
is live streaming of TV linear output, or live streaming such as Twitch or
Facebook, or non-UGC services like OTT offerings made by broadcasters.

# Requirements {#requirements}

Even a cursory examination of the existing proposals listed in {{priorart}} shows that there are fundamental differences in the approaches being used - for instance, whether a proposal uses RTP as its Media Transport Protocol.

In this section, we attempt to focus on high-level requirements for real time media streaming over a QUIC connection, recognizing that

* additional analysis will be required, and

* we are starting with requirements that are apparent for RTP-based proposals

## Codec Agility

When initiating a media session, both the sender and receiver should be able to
negotiate the codecs, bitrates and other media details based on capabilities and
preferences.

## Support a range of Latencies

TODO: confirm requirements for latency

{{I-D.draft-ietf-mops-streaming-opcons}} describes these latency requirements for streaming media.

- ultra low-latency (less than 1 second)
- low-latency live (less than 10 seconds)
- non-low-latency live (10 seconds to a few minutes)
- on-demand (hours or more)

## Congestion Control

TODO: Confirm these requirements, consider looking at how RFC 8836 applies to
this requirement.

## Support Lossless and Lossy Media Transport

TODO: confirm scope of this draft to describe lossless media transport, lossy media transport, or both lossless and lossy transport.

## Flow Directionality

Media should be able to flow in either direction from client to server or
vice-versa, either individually or concurrently.

## WebTransport

TODO: Unsure if this should be a requirement. If it is, we have to consider two
things: WebTransport supports HTTP/2, are we going to explicitly exclude it?
Also, WebTransport {{I-D.draft-ietf-webtrans-overview}} has normative language
around congestion control which may be at odds with our potential requirements.

## Authentication

The encapsulation SHOULD have capabilities beyond what QUIC provides to allow hosts
to authenticate one another, this should be kept simple but robust in nature to
prevent attacks like credential brute-forcing.

TODO: More details are required here

# Non-requirements

This section covers topics that are explicitly out of scope for the time being.

## NAT Traversal

From Section 8.2 of {{RFC9000}}:

> Path validation is not designed as a NAT traversal mechanism. Though the mechanism described here might be effective for the creation of NAT bindings that support NAT traversal, the expectation is that one endpoint is able to receive packets without first having sent a packet on that path. Effective NAT traversal needs additional synchronization mechanisms that are not provided here.

Although there are use cases that would benefit from a mechanism for NAT traversal, a QUIC protocol extention would be required to support those use cases today.

## New Media Transport Protocols

The creation of new media transport protocols should be avoided, and instead we
should make use of RTP {{RFC3550}} and the existing ecosystem of payload formats
and methods of signalling where possible. Work on QUIC encapsulation may reveal a need to extend
these specificiations; in which case we should work with the relevant working
groups and present our use-cases.

## Multicast

Even if multicast and other network broadcasting capabilities are often used in delivering media in our use cases, QUIC doesn't yet support multicast, and would require a QUIC protocol extension to do so. In addition, the inclusion of multicast would introduce more complexity in both the specification and
client implimentations.

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to guide discussion and consensus, it introduces
no security considerations of its own.

--- back

# Acknowledgements

The authors would like the thank the many authors of of the specifications referenced in {{priorart}} for their work:

* Alan Frindell
* Colin Perkins
* Jake Weissman
* Joerg Ott
* Jordi Cenzano
* Kirill Pugin
* Maria Sharabayko
* Mathis Engelbart
* Maxim Sharabayko
* Roni Even
* Sam Hurst
* Varun Singh

James Gruessing would also like to thank Francesco Illy and Nicholas Book for their part in providing the needed motivation.
