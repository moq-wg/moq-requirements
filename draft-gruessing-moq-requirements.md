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
  AVTCORE-2022-02:
    target: https://datatracker.ietf.org/meeting/interim-2022-avtcore-01/session/avtcore
    title: "AVTCORE 2022-02 interim meeting materials"
    date: February 2022
  MOQ-ml:
    target: https://www.ietf.org/mailman/listinfo/moq
    title: "Moq -- Media over QUIC"
  DASH:
    target: https://www.iso.org/standard/79329.html
    title: "ISO/IEC 23009-1:2019: Dynamic adaptive streaming over HTTP (DASH) -- Part 1: Media presentation description and segment formats (2nd edition)"
  ISOBMFF:
    target: https://www.iso.org/standard/83102.html
    title: "ISO/IEC 14496-12:2022 Information technology — Coding of audio-visual objects — Part 12: ISO base media file format"
    date:  January 2022
  rtcweb:
    target: https://datatracker.ietf.org/wg/rtcweb/about/
    title: "Real-Time Communication in WEB-browsers (rtcweb) IETF Working Group"
  QUIC-goals:
    target: https://datatracker.ietf.org/doc/charter-ietf-quic/01/
    title: "Initial Charter for QUIC Working Group"
    date: October 4, 2016
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-kpugin-rush:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-sharabayko-srt:
  I-D.draft-ietf-quic-http:
  I-D.draft-ietf-quic-multipath:
  I-D.draft-lcurley-warp:
  RFC3550:
  RFC8216:

--- abstract

This document describes the use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", and recommends use cases on live media contribution, syndication, and streaming as the basis for discussions that should guide the design of protocols to satisfy these use cases.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MoQ)
mailing list, at <https://www.ietf.org/mailman/listinfo/moq>.

--- middle

# Introduction {#intro}

This document describes the use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", and recommends use cases on live media contribution, syndication, and streaming as the basis for requirements discussions that should guide the design of protocols to satisfy these use cases.

## For The Impatient Reader

This document is intended to report a survey of use cases that have been discussed under the "Media Over QUIC" banner, and to propose a subset of those use cases that should be considered first.

Our proposal is in {{propscope}}, our understanding of the requirements for those use cases is in {{requirements}}, and most of the rest of the document provides background for those sections.

## Why QUIC For Media? {#why-quic}

It is not the purpose of this document to argue against proposals for work on media applications that do not involve QUIC. Such proposals are simply out of scope for this document.

When work on the QUIC protocol ({{RFC9000}}) was chartered ({{QUIC-goals}}), the key goals for QUIC were:

- Minimizing connection establishment and overall transport latency for applications, starting with HTTP/2,
- Providing multiplexing without head-of-line blocking,
- Requiring only changes to path endpoints to enable deployment,
- Enabling multipath and forward error correction extensions, and
- Providing always-secure transport, using TLS 1.3 by default.

These goals were chosen with HTTP ({{I-D.draft-ietf-quic-http}}) in mind.

While work on "QUIC version 1" (version codepoint 0x00000001) was underway, protocol designers considered potential advantages of the QUIC protocol for other applications. In addition to the key goals for HTTP applications, these advantages were immediately apparent for at least some media applications:

- QUIC endpoints can create bidirectional or unidirectional ordered byte streams.
- QUIC will automatically handle congestion control, packet loss, and reordering for stream data.
- QUIC streams allow multiple media streams to share congestion and flow control without otherwise blocking each other.
- QUIC streams also allow partial reliability, since either the sender or receiver can terminate the stream early without affecting the overall connection.
- With the DATAGRAM extension ({{I-D.draft-ietf-quic-datagram}}), further partially reliable models are possible, and applications can send congestion controlled datagrams below the MTU size.
- QUIC connections are established using an ALPN.
- QUIC endpoints can choose and change their connection ID.
- QUIC endpoints can migrate IP address without breaking the connection.
- Because QUIC is encapsulated in UDP, QUIC implementations can run in user space, rather than in kernel space, as TCP typically does. This allows more room for extensible APIs between application and transport, allowing more rapid implementation and deployment of new congestion control, retransmission, and prioritization mechanisms.
- QUIC is supported in browsers via HTTP/3 or WebTransport.
- With WebTransport, it is possible to write libraries or applications in JavaScript.

# Terminology {#term}

## The Many Meanings of "Media Over QUIC" {#moq-meaning}

Protocol developers have been considering the implications of the QUIC protocol ({{RFC9000}}) for media transport for several years, resulting in a large number of possible meanings of the term "Media Over QUIC", or "MOQ". As of this writing, "Media Over QUIC" has had at least these meanings:

- any kind of media carried directly over the QUIC protocol, as a QUIC payload
- any kind of media carried indirectly over the QUIC protocol, as an RTP payload ({{RFC3550}})
- any kind of media carried indirectly over the QUIC protocol, as an HTTP/3 payload
- any kind of media carried indirectly over the QUIC protocol, as a WebTransport payload
- the encapsulation of any Media Transport Protocol ({{mtp}}) in a QUIC payload
- an IETF mailing list ({{MOQ-ml}}) "... for discussion of video ingest and distribution protocols that use QUIC as the underlying transport"

There may be IETF participants using other meanings as well.

As of this writing, the second bullet ("any kind of media carried indirectly over the QUIC protocol, as an RTP payload"), seems to be in scope for the IETF AVTCORE working group, and was discussed at some length at the February 2022 AVTCORE working group meeting {{AVTCORE-2022-02}}. So, perhaps, that possible meaning is out of scope for "Media over QUIC".

It will be SUPER HELPFUL if interested parties can come up with a term that unambiguously describes what we're trying to achieve.

## Media Transport Protoccol {#mtp}

This document describes considerations for work on a new "Network Transport Protocol", or possibly, extensions to an existing "Network Transport Protocol".

Within this document, we use the term "Media Transport Protocol" to describe the protocol of interest. This is easier to understand if the reader assumes that we are talking about a protocol stack that looks something like this:

~~~~~~
               Media
    ---------------------------
           Media Format
    ---------------------------
    Media Transport Protocol(s)
    ---------------------------
               QUIC
~~~~~~

where "Media Format" would be something like RTP payload formats or ISOBMFF {{ISOBMFF}}, and "Media Transport Protocol" would be something like RTP or HTTP. Not all of the possible proposals for "Media Over QUIC" follow this model, but for the ones that do, it seems useful to have names for "the protocol layers beteern Media and QUIC".

It is worth noting explicitly that the "Media Transport Protocol" layer might include more than one protocol. For example, a new Media Transport Protocol might be defined to run over HTTP, or even over WebTransport, which would imply HTTP as well.

## Latency Requirement Categories {#latent-cat}

Within this document, we extend the latency requirement categories for streaming media described in {{I-D.draft-ietf-mops-streaming-opcons}}:

- ultra low-latency (less than 1 second)
- low-latency live (less than 10 seconds)
- non-low-latency live (10 seconds to a few minutes)
- on-demand (hours or more)

These latency bands were appropriate for streaming media, which was the target for {{I-D.draft-ietf-mops-streaming-opcons}}, but some realtime media may have requirements that are significantly less than "ultra-low latency". Within this document, we are also using

- ull500 (less than 500 ms)
- ull100 (less than 100 ms)

Obviously, these last two latency bands are the shortened form of "ultra-low latency - 500 ms" and "ultra-low-latency - 100 ms". Also obviously, bikeshedding on better names is welcomed.

# Prior and Existing Specifications {#priorart}

* Note - need to edit this section to reflect new draft scope and add short characterization of WARP protocol.

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

Whilst RUSH predates the datagram specification, it uses its own frame types on
top of QUIC to take advantage of QUIC implementations reassembling messages
greater than MTU. In addition individual media frames are given their own stream
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

## Warp - Segmented Live Video Transport {#warp}

{{I-D.draft-lcurley-warp}}

Warp's specification attemps to map Group of Picture encoding of video on top of
QUIC streams. It depends on ISOBMFF containers to encapsulate both media as well
as messaging, and defines prioritisation with separate considerations for audio
and video. It doesn't yet define bi-directionality of media flows, and can be
run over protocols like WebTransport {{I-D.draft-ietf-webtrans-overview}}.

## Comparison of Existing Specifications

* Both QRT and the Engelbart draft attempt to use existing payloads of RTP,
  RTCP, and SDP, unlike RUSH and SRT, as well as using existing Datagram frames
* All drafts take differing approaches to flow/stream identification and
  management; some address congestion control and others just omit the subject
  and leave it to QUIC to handle
* Both QRT and RUSH specify ALPN identification; the Engelbart, Warp, and SRT
  drafts do not.


## Moving Beyond "RTP over QUIC".

It's worth noting that work on "RTP over QUIC" is being considered in the AVTCORE working group at this time, although no proposals have been adopted by the working group.

Although some of the use cases described in {{overallusecases}} came out of "RTP over QUIC" proposals, they are worth considering for MOQ, and may be especially relevant to MOQ, depending on whether "RTP over QUIC' requires major changes to RTP and RTCP, in order to meet the requirements arising out of those use cases.

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that have been proposed for "Media Over QUIC".

An early draft in the "media over QUIC" space,
{{I-D.draft-rtpfolks-quic-rtp-over-quic}}, defined several key use cases. Some of the
following use cases have been inspired by that document, and others have come from discussions with the
wider MOQ community (among other places, a side meeting at IETF 112).

For each use case in this section, we also define

* the number of senders or receiver in a given session transmitting distinct streams,
* whether a session has bi-direction flows of media from senders and receivers, and
* the expected lowest latency requirements using the definitions specified in {{term}}.

It is likely that we should add other characteristics, as we come to understand them.

## Video Conferencing {#vidconf}

**Senders/Receivers**: Many to Many
**Bi-directional**: Yes
**Latency**: Ultra-Low

Where media is both sent and received; This may include audio from both
microphone(s) or other inputs, or may include "screen sharing" or inclusion of
other content such as slide, document, or video presentation. This may be done
as client/server, or peer to peer with a many to many relationship of both
senders and receivers.

## Gaming {#gaming}

**Senders/Receivers**: One to One
**Bi-directional**: Yes
**Latency**: Ull100

Where media is received, and user inputs are sent by the client. This may also
include the client receiving other types of signalling, such as triggers for
haptic feedback. This may also carry media from the client such as microphone
audio for in-game chat with other players.

## Remote Desktop {#remdesk}

**Senders/Receivers**: One to One
**Bi-directional**: Yes
**Latency**: Ultra-Low

Where media is received, and user inputs are sent by the client. Latency
requirements with this usecase are marginally different than the gaming use
case. This may also include signalling and/or transmitting of files or devices
connected to the user's computer.

## Live Media Streaming {#lmstream}

**Senders/Receivers**: One to Many
**Bi-directional**: No
**Latency**: Low to Non-Low

Where media is received from a live broadcast or stream. This may comprise of
multiple audio or video outputs with different codecs or bitrates. This may also
include other types of media essence such as subtitles or timing signalling
information (e.g. markers to indicate change of behaviour in client such as
advertisement breaks). The use of "live rewind" where a window of media behind
the live edge can be made available for clients to playback, either because the
local player falls behind edge or because the viewer wishes to play back from a
point in the past.

## Live Media Contribution {#lmcont}

**Senders/Receivers**: One to One
**Bi-directional**: No
**Latency**: Ultra-Low to Low

Where media is received from a source for onwards handling into a distribution
platform. The media may comprise of multiple audio and/or video sources.
Bitrates may either be static or set dynamically by signalling of connection
inforation (bandwidth, latency) based on data sent by the receiver.

## Live Media Syndication {#lmsynd}

**Senders/Receivers**: One to One
**Bi-directional**: No
**Latency**: Ultra-Low to Low

Where media is sent onwards to another platform for further distribution. The
media may be compressed down to a bitrate lower than source, but larger than
final distribution output. Streams may be redundant with failover mechanisms in
place.

## On-Demand Media Streaming {#odstream}

**Senders/Receivers**: One to Many
**Bi-directional**: No
**Latency**: On Demand

Where media is received from a non-live, typically pre-recorded source. This may
feature additional outputs, bitrates, codecs, and media types described in the
live media streaming use case.

# Proposed Scope for "Media Over QUIC" {#propscope}

Our proposal is that "Media Over QUIC" discussions focus first on the use cases for Live Media Contribution ({{lmcont}}),
Syndication ({{lmsynd}}), and Streaming ({{lmstream}}). Our reasoning is provided in {{usecaseanalysis}}.

## Use Case Analysis {#usecaseanalysis}

Each of the above use cases in {{overallusecases}} fit into one of three classifications of solutions.

The first group, covering gaming ({{gaming}}), screen sharing ({{remdesk}}), and general video conferencing ({{vidconf}}), are
largely covered by WebRTC and related protocols today. Whilst there may be
benefit in these use cases having a QUIC based protocol it may be more
appropriate given the size of existing deployments to extend the WebRTC
protocol and specifications. Such work could start in a QUIC specific forum, but
would likely need to take place in {{rtcweb}} and the W3C.

The second group of classifications, covering Live Media Contribution ({{lmcont}}),
Syndication ({{lmsynd}}), and Streaming ({{lmstream}}) are likely the use cases likely to benefit most from
this work. Existing protocols used such as HLS {{RFC8216}} and DASH {{DASH}}
are reaching limits towards how low they can reduce latency in live streaming
and for scenarios where low-bitrate audio streams are used add a significant
amount of overheads compared to the media bitstream.

On-Demand media streaming ({{odstream}}) is unlikely to benefit from work in
this space. Without notable latency requirements, protocols such as HLS and
DASH largely meet the needs of this use case.

# Requirements {#requirements}

**Note: This section was written to reflect an early focus on "Media over RTP over QUIC", and will be revisited when we agree on the use cases in {{overallusecases}} that will be in scope for further work.

Even a cursory examination of the existing proposals listed in {{priorart}}
shows that there are fundamental differences in the approaches being used.

## Codec Agility

When initiating a media session, both the sender and receiver should be able to
negotiate the codecs, bitrates, resolution, and other media details based on
capabilities and preferences. This must be negotiable both before commencing
playback but also during as a result of changes to device output or network
conditions (such as reduction in available network bandwidth). It may be
prefered to use existing ecosystem for such purposes, e.g. SDP {{RFC4566}}.

## Support a range of Latencies

Support for a nominal latency appropriate for the use cases that are in scope should be
achieved, with consideration for the minimum buffer that a receiver playing content may
need to handle congestion, packet loss, and other degradation in network
quality.

## Migration of Sessions

Handling of migration of a session between hosts, either of sender or receiver
should be supported. This may either happen because the sender is undergoing
maintenence or a rebalancing of resource, because the either is experiencing a
change in network connectivity (such as a device moving from WiFi to cellular
connectivity) or other reasons.

This may depend on QUIC capabilities such as {{I-D.draft-ietf-quic-multipath}}
but is by no means a hard requirement.

## Congestion Control

TODO: Confirm these requirements, consider looking at how RFC 8836 applies to
this requirement.

## Support Lossless and Lossy Media Transport

TODO: confirm scope of this draft to describe lossless media transport, lossy media transport, or both lossless and lossy transport.

## Flow Directionality

Media should be able to flow in either direction from client to server or
vice-versa, either individually or concurrently but should only be negotiated at
the start of the session.

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

The authors would like to thank the many authors of the specifications referenced in {{priorart}} for their work:

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

The authors would like to thank Alan Frindell, Luke Curley, and Maxim Sharabayko for text contributions to this draft.

James Gruessing would also like to thank Francesco Illy and Nicholas Book for their part in providing the needed motivation.
