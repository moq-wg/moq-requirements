---
title: Media Over QUIC - Use cases and Requirements
abbrev: MoQ Use Cases and Requirements
docname: draft-gruessing-moq-requirements-latest
category: info
date:

ipr: trust200902
area: applications
workgroup: MOQ Mailing List
keyword: Internet-Draft QUIC
stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, subcompact, comments, inline]

author:
  -
    ins: J. Gruessing
    name: James Gruessing
    organization: Nederlandse Publieke Omroep
    country: Netherlands
    email: james.ietf@gmail.com
  -
    ins: S. Dawkins
    name: Spencer Dawkins
    organization: Tencent America LLC
    country: United States of America
    email: spencerdawkins.ietf@gmail.com

normative:
  RFC4566:
  RFC9000:
  I-D.draft-ietf-webtrans-overview:
  I-D.draft-ietf-mops-streaming-opcons:
  I-D.draft-ietf-quic-datagram:

informative:
  DASH:
    target: https://www.iso.org/standard/79329.html
    title: "ISO/IEC 23009-1:2019: Dynamic adaptive streaming over HTTP (DASH) -- Part 1: Media presentation description and segment formats (2nd edition)"
  rtcweb:
    target: https://datatracker.ietf.org/wg/rtcweb/about/
    title: Real-Time Communication in WEB-browsers (rtcweb) IETF Working Group
  I-D.draft-dawkins-sdp-rtp-quic-questions:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-kpugin-rush:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-sharabayko-srt:
  I-D.draft-ietf-quic-http:
  RFC3550:
  RFC8216:

--- abstract

This document describes the uses cases, requirements, and considerations that should guide the design of the encapsulation of a real-time media transport protocol as a payload in the QUIC protocol, that will be used for live media contribution, syndication, and streaming.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MOQ) mailing list, at https://www.ietf.org/mailman/listinfo/moq.

--- middle

# Introduction {#intro}

This document describes the uses cases, requirements, and considerations that should guide the design of the encapsulation of a real-time media transport protocol as a payload in the QUIC protocol ({{RFC9000}}), that will be used for live media contribution, syndication, and streaming.

Protocol developers have been considering the implications of the QUIC protocol ({{RFC9000}}) on media transport for several years, but the initial focus on QUIC in the IETF was to support web applications that used the HTTP/3 protocol {{I-D.draft-ietf-quic-http}}. The completion of the initial versions of the QUIC specifications, and the adoption of {{I-D.draft-ietf-quic-datagram}}, have cleared the way for proposals to use QUIC as a media transport. This document considers a number of proposals for "Media Over QUIC", and analyzes them to understand requirements and considerations.

# Terminology {#term}

Within this document, we use the term "Media Transport Protocol". This is easier to understand if the reader assumes that we are starting with a protocol stack that looks like this:

~~~~~~
            Media
    ------------------------
    Media Transport Protocol
~~~~~~

and the goal is to provide a protocol stack that looks like this:

~~~~~~
            Media
    ------------------------
    Media Transport Protocol
    ------------------------
             QUIC
~~~~~~

Not all of the proposals for "Media Over QUIC" follow this model, but for the ones that do, it seems useful to have a name for "the protocol layer immediately beneath media".

# Prior and Existing Specifications {#priorart}

* Note - need to edit the following paragraph to reflect new draft scope

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
between the two protocols. Apart from that, SRT provides a native suport for stream multiplexing,
thus contributing this missing functionality to QUIC datagrams.

## Comparison of Existing Specifications

* Both QRT and the Engelbart draft attempt to use existing payloads of RTP,
  RTCP, and SDP, unlike RUSH and SRT, as well as using existing Datagram frames
* RUSH introduces new frame types as its development pre-dates Datagram frames
* All drafts take differing approaches to flow/stream identification and
  management; some address congestion control and others just omit the subject
  and leave it to QUIC to handle
* Both QRT and RUSH specify ALPN identification; the Engelbart and SRT drafts do not.

## Moving Beyond "RTP over QUIC".

It's worth noting that work on "RTP over QUIC" is being considered in the AVTCORE working group at this time, although no proposals have been adopted by the working group.

Although some of the use cases described in {{usecases}} came out of "RTP over QUIC" proposals, they are worth considering for MOQ, depending on whether "RTP over QUIC' requires major changes to RTP and RTCP, in order to meet the requirements arising out of those use cases.

# Use Cases {#usecases}

## Use Cases From {{I-D.draft-rtpfolks-quic-rtp-over-quic}}

An early draft in the "media over QUIC" space,
{{I-D.draft-rtpfolks-quic-rtp-over-quic}}, defined several key use cases. The
following are some inspired from that document or from discussions with the
wider community. For each we also define the number of senders or receiver in a
given session transmitting distinct streams, if a session has bi-direction flows
of media from senders and receivers, and the expected lowest latency
requirements using the definitions specified in
{{I-D.draft-ietf-mops-streaming-opcons}}.

## Video Conferencing

**Senders/Receivers**: Many to Many
**Bi-directional**: Yes
**Latency**: Ultra-Low

Where media is both sent and received; This may include audio from both
microphone(s) or other inputs, or may include "screen sharing" or inclusion of
other content such as slide, document, or video presentation. This may be done
as client/server, or peer to peer with a many to many relationship of both
senders and receivers.

## Gaming

**Senders/Receivers**: One to One
**Bi-directional**: Yes
**Latency**: Sub-Ultra-Low

Where media is received, and user inputs are sent by the client. This may also
include the client receiving other types of signalling, such as triggers for
haptic feedback. This may also carry media from the client such as microphone
audio for in-game chat with other players.

## Remote Desktop

**Senders/Receivers**: One to One
**Bi-directional**: Yes
**Latency**: Ultra-Low

Where media is received, and user inputs are sent by the client. Latency
requirements with this usecase are marginally different than the gaming use
case. This may also include signalling and/or transmitting of files or devices
connected to the user's computer.

## Live Media Streaming

**Senders/Receivers**: One to Many
**Bi-directional**: No
**Latency**: Low to Non-Low

Where media is received from a live broadcast or stream. This may comprise of
multiple audio or video outputs with different codecs or bitrates. This may also
include other types of media essence such as subtitles or timing signalling
information (e.g. markers to indicate change of behaviour in client such as
advertisement breaks)

## Live Media Contribution

**Senders/Receivers**: One to One
**Bi-directional**: No
**Latency**: Ultra-Low to Low

Where media is received from a source for onwards handling into a distribution
platform. The media may comprise of multiple audio and/or video sources.
Bitrates may either be static or set dynamically by signalling of connection
inforation (bandwidth, latency) based on data sent by the receiver.

## Live Media Syndication

**Senders/Receivers**: One to One
**Bi-directional**: No
**Latency**: Ultra-Low to Low

Where media is sent onwards to another platform for further distribution. The
media may be compressed down to a bitrate lower than source, but larger than
final distribution output. Streams may be redundant with failover mechanisms in
place.

## On-Demand Media Streaming

**Senders/Receivers**: One to Many
**Bi-directional**: No
**Latency**: On Demand

Where media is received from a non-live, typically pre-recorded source. This may
feature additional outputs, bitrates, codecs, and media types described in the
live media streaming use case.

# Suggested Use Cases for "Media Over QUIC"

--- note_Note_to_Readers

This section is a work in progress, and is based on the opinions of the draft
authors. We are happy to be guided by discussion about other use cases.

Each of the above use cases fit into three classifications of solutions, with
the first three covering gaming, screen sharing, and general video conferencing
largely covered by WebRTC and related protocols today. Whilst there may be
benefit in these use cases having a QUIC based protocol it may be more
appropriate given the size of existing deployments to extend the WebRTC
protocol and specifications. Such work could start in a QUIC specific forum, but
would likely need to take place in {{rtcweb}} and the W3C.

The second group of classifications covering Live Media Contribution,
Syndication, and Streaming are likely the use cases likely to benefit most from
this work. Existing protocols used such as HLS {{RFC8216}} and DASH {{DASH}}
are reaching limits towards how low they can reduce latency in live streaming
and for scenarios where low-bitrate audio streams are used add a significant
amount of overheads compared to the media bitstream.

On-Demand media streaming is unlikely to benefit from work in this space,
without notable latency requirements and protocols such as HLS and DASH meeting
the needs of this use case.

# Requirements {#requirements}

Even a cursory examination of the existing proposals listed in {{priorart}} shows that there are fundamental differences in the approaches being used - for instance, whether a proposal uses RTP as its Media Transport Protocol.

In this section, we attempt to focus on high-level requirements for real time media streaming over a QUIC connection, recognizing that

* additional analysis will be required, and

* we are starting with requirements that are apparent for RTP-based proposals

## Codec Agility

When initiating a media session, both the sender and receiver should be able to
negotiate the codecs, bitrates and other media details based on capabilities and
preferences.
It may be prefered to use existing ecosystem for such purposes, e.g. SDP {{RFC4566}}.

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
On the other hand, UDP multicast may be considered as the last mile delivery transport outside of QUIC transport, thus
it would be beneficial for a protocol to provide such an opportunity (e.g. RTP/QUIC -> RTP/UDP).

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

The authors would like to thank Maxim Sharabayko for text contributions to this draft.

James Gruessing would also like to thank Francesco Illy and Nicholas Book for their part in providing the needed motivation.
