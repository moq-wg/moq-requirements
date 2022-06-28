---
title: Media Over QUIC - Use Cases and Considerations for Media Transport Protocol Design
abbrev: MoQ Use Cases and Considerations
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

informative:
  RFC3550:
  RFC4566:
  RFC6363:
  RFC6716:
  RFC7230:
  RFC7540:
  RFC7826:
  RFC8216:
  RFC8298:
  RFC8312:
  RFC8836:
  RFC9000:
  RFC9002:
  RFC8698:

  I-D.draft-cardwell-iccrg-bbr-congestion-control:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-kpugin-rush:
  I-D.draft-lcurley-warp:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-sharabayko-srt:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-jennings-moq-quicr-arch:
  I-D.draft-jennings-moq-quicr-proto:

  I-D.draft-ietf-mops-streaming-opcons:
  I-D.draft-ietf-quic-datagram:
  I-D.draft-ietf-quic-http:
  I-D.draft-ietf-quic-multipath:
  I-D.draft-ietf-webtrans-overview:

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
  WebRTC:
    target: https://www.w3.org/groups/wg/webrtc
    title: "Web Real-Time Communications Working Group"
  QUIC-goals:
    target: https://datatracker.ietf.org/doc/charter-ietf-quic/01/
    title: "Initial Charter for QUIC Working Group"
    date: October 4, 2016

--- abstract

This document describes use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", provides analysis about those use cases, recommends a subset of use cases that cover live media ingest, syndication, and streaming for further exploration, and describes considerations that should guide the design of protocols to satisfy these use cases.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MoQ)
mailing list, at <https://www.ietf.org/mailman/listinfo/moq>.

--- middle

# Introduction {#intro}

This document describes use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", provides analysis about those use cases, recommends a subset of use cases that cover live media ingest, syndication, and streaming for further exploration, and describes considerations that should guide the design of protocols to satisfy these use cases.

## For The Impatient Reader

- Our proposal is to focus on live media use cases, as described in {{propscope}}, rather than on interactive media use cases or on-demand use cases.
- The reasoning behind this proposal can be found in {{analy-interact}}.
- The considerations for protocol work to satisfy the proposed use cases can be found in {{considerations}}.

Most of the rest of this document provides background for these sections.

## Why QUIC For Media? {#why-quic}

It is not the purpose of this document to argue against proposals for work on media applications that do not involve QUIC. Such proposals are simply out of scope for this document.

When work on the QUIC protocol ({{RFC9000}}) was chartered ({{QUIC-goals}}), the key goals for QUIC were:

- Minimizing connection establishment and overall transport latency for applications, starting with HTTP,
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

The specific advantages of interest may vary from use case to use case, but these advantages justify further investigation of "Media Over QUIC".

# Terminology {#term}

## The Many Meanings of "Media Over QUIC" {#moq-meaning}

Protocol developers have been considering the implications of the QUIC protocol ({{RFC9000}}) for media transport for several years, resulting in a large number of possible meanings of the term "Media Over QUIC", or "MOQ". As of this writing, "Media Over QUIC" has had at least these meanings:

- any kind of media carried directly over the QUIC protocol, as a QUIC payload
- any kind of media carried indirectly over the QUIC protocol, as an RTP payload ({{RFC3550}})
- any kind of media carried indirectly over the QUIC protocol, as an HTTP/3 payload
- any kind of media carried indirectly over the QUIC protocol, as a WebTransport payload
- the encapsulation of any Media Transport Protocol ({{mtp}}) in a QUIC payload
- an IETF mailing list ({{MOQ-ml}}), which was requested "... for discussion of video ingest and distribution protocols that use QUIC as the underlying transport", although discussion of other Media Over QUIC proposals have also been discussed there.

There may be IETF participants using other meanings as well.

As of this writing, the second bullet ("any kind of media carried indirectly over the QUIC protocol, as an RTP payload"), seems to be in scope for the IETF AVTCORE working group, and was discussed at some length at the February 2022 AVTCORE working group meeting {{AVTCORE-2022-02}}, although no drafts in this space have yet been adopted by the AVTCORE working group.

## Media Transport Protoccol {#mtp}

This document describes considerations for work on extensions to existing "Media Transport Protocols" or creation of new "Media Transport Protocols".

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

where "Media Format" would be something like RTP payload formats or ISOBMFF {{ISOBMFF}}, and "Media Transport Protocol" would be something like RTP or HTTP. Not all possible proposals for "Media Over QUIC" follow this model, but for the ones that do, it seems useful to have names for "the protocol layers beteern Media and QUIC".

It is worth noting explicitly that the "Media Transport Protocol" layer might include more than one protocol. For example, a new Media Transport Protocol might be defined to run over HTTP, or even over WebTransport and HTTP.

## Latency Requirement Categories {#latent-cat}

Within this document, we extend the latency requirement categories for streaming media described in {{I-D.draft-ietf-mops-streaming-opcons}}:

- ultra low-latency (less than 1 second)
- low-latency live (less than 10 seconds)
- non-low-latency live (10 seconds to a few minutes)
- on-demand (hours or more)

These latency bands were appropriate for streaming media, which was the target for {{I-D.draft-ietf-mops-streaming-opcons}}, but some interactive media may have requirements that are significantly less than "ultra-low latency". Within this document, we are also using

- near real-time (less than 50 ms)
- Ull-200 (less than 200 ms)

Perhaps obviously, these last two latency bands are the shortened form of "ultra-low latency - 50 ms" and "ultra-low-latency - 200 ms". Perhaps less obviously, bikeshedding on better names and more useful values is welcomed.

# Prior and Existing Specifications {#priorart}

Several draft specifications have been proposed which either encapsulate existing Media Transport Protocols in QUIC, or define
their own new Media Transport Protocol on top of QUIC. With the exception of RUSH ({{kpugin}}), it is unknown if the other specifications listed in this section
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
primarily for ingest transport use cases and this specification covers the
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

## QuicR {#quicr}

{{I-D.draft-jennings-moq-quicr-proto}}

QuicR comprisees of both an architectural document
{{I-D.draft-jennings-moq-quicr-arch}} as well as a protocol specification.
QuicR's focus is around Pub/Sub patterns for both client negotiation as well as
relaying media and allows implementations to use either streams or datagrams to
carry media, and whilst it does not yet appear to define a means of media
encapsulation, it does appear possible for QuicR to overlay support for both
{{kpugin}} and {{warp}}.

## Comparison of Existing Specifications

** Additional details for this comparison could usefully be added here. **

* Some drafts attempt to use existing payloads of RTP, RTCP, and SDP, while others do not.
* Some use QUIC Datagram frames, while others use QUIC streams.
* All drafts take differing approaches to flow/stream identification and management. Some address congestion control and others just defer this to QUIC to handle.
* Some drafts specify ALPN identification, while others do not.

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that have been proposed for "Media Over QUIC".

Although some of the use cases described in this section came out of "RTP over QUIC" proposals, they are worth considering in the broader "Media Over QUIC" context, and may be especially relevant to MOQ, depending on whether "RTP over QUIC" requires major changes to RTP and RTCP, in order to meet the requirements arising out of the corresponding use cases.

An early draft in the "media over QUIC" space,
{{I-D.draft-rtpfolks-quic-rtp-over-quic}}, defined several key use cases. Some of the
following use cases have been inspired by that document, and others have come from discussions with the
wider MOQ community (among other places, a side meeting at IETF 112).

For each use case in this section, we also define

* the number of senders or receiver in a given session transmitting distinct streams,
* whether a session has bi-direction flows of media from senders and receivers, and
* the expected lowest latency requirements using the definitions specified in {{term}}.

It is likely that we should add other characteristics, as we come to understand them.

## Interactive Media {#interact}

The use cases described in this section have one particular attribute in common - the target latency for these cases are on the order of one or two RTTs. In order to meet those targets, it is not possible to rely on protocol mechanisms that require multiple RTTs to function effectively. For example,

* When the target latency is on the order of one RTT, it makes sense to use FEC {{RFC6363}} and codec-level packet loss concealment {{RFC6716}}, rather than selectively retransmitting only lost packets. These mechanisms use more bytes, but do not require multiple RTTs in order to recover from packet loss.
* When the target latency is on the order of one RTT, it is impossible to use congestion control schemes like BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}}, since BBR has probing mechanisms that rely on temporarily inducing delay and amortizing the consequences of that over multiple RTTs.

This may help to explain why these use cases often rely on protocols such as RTP {{RFC3550}}, which provide low-level control of packetization and transmission.

### Gaming {#gaming}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| Yes
| **Latency**|  Ull-50

Where media is received, and user inputs are sent by the client. This may also
include the client receiving other types of signalling, such as triggers for
haptic feedback. This may also carry media from the client such as microphone
audio for in-game chat with other players.

### Remote Desktop {#remdesk}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| Yes
| **Latency**|  Ull-50

Where media is received, and user inputs are sent by the client. Latency
requirements with this usecase are marginally different than the gaming use
case. This may also include signalling and/or transmitting of files or devices
connected to the user's computer.

### Video Conferencing/Telephony {#vidconf}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  Many to Many
| **Bi-directional**| Yes
| **Latency**|  Ull-50 to Ull-200

Where media is both sent and received; This may include audio from both
microphone(s) or other inputs, or may include "screen sharing" or inclusion of
other content such as slide, document, or video presentation. This may be done
as client/server, or peer to peer with a many to many relationship of both
senders and receivers. The target for latency may be as large as Ull-200 for
some media types such as audio, but other media types in this use case have much
more stringent latency targets.

## Hybrid Interactive and Live Media

For the video conferencing/telephony use case, there can be additional scenarios
where the audience greatly outnumbers the concurrent active participants, but
any member of the audience could participate. As this has a much larger total
number of participants - as many as Live Media Streaming {{lmstream}}, but with
the bi-directionality of confercing, this should be considered a "hybrid".

## Live Media {#lm-media}

The use cases in this section, unlike the use cases described in {{interact}}, still have "humans in the loop", but these humans expect media to be "responsive", where the responsiveness is more on the order of 5 to 10 RTTs. This allows the use of protocol mechanisms that require more than one or two RTTs - as noted in {{interact}}, end-to-end recovery from packet loss and congestion avoidance are two such protocol mechanisms that can be used with Live Media.

To illustrate the difference, the responsiveness expected with videoconferencing is much greater than watching a video, even if the video is being produced "live" and sent to a platform for syndication and distribution.

### Live Media Ingest {#lmingest}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| No
| **Latency**|  Ull-200 to Ultra-Low

Where media is received from a source for onwards handling into a distribution
platform. The media may comprise of multiple audio and/or video sources.
Bitrates may either be static or set dynamically by signalling of connection
inforation (bandwidth, latency) based on data sent by the receiver.

### Live Media Syndication {#lmsynd}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| No
| **Latency**|  Ull-200 to Ultra-Low

Where media is sent onwards to another platform for further distribution. The
media may be compressed down to a bitrate lower than source, but larger than
final distribution output. Streams may be redundant with failover mechanisms in
place.

### Live Media Streaming {#lmstream}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
| **Bi-directional**| No
| **Latency**| Ull-200 to Ultra-Low

Where media is received from a live broadcast or stream. This may comprise of
multiple audio or video outputs with different codecs or bitrates. This may also
include other types of media essence such as subtitles or timing signalling
information (e.g. markers to indicate change of behaviour in client such as
advertisement breaks). The use of "live rewind" where a window of media behind
the live edge can be made available for clients to playback, either because the
local player falls behind edge or because the viewer wishes to play back from a
point in the past.

## On-Demand Media {#od-media}

Finally, the "On-Demand" use cases described in this section do not have a tight linkage between ingest and streaming, allowing significant transcoding, processing, insertion of video clips in a news article, etc. The latency constraints for the use cases in this section may be dominated by the time required for whatever actions are required before media are available for streaming.

### On-Demand Ingest {#od-ingest}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
| **Bi-directional**| No
| **Latency**|  On Demand

Where media is ingested and processed for a system to later serve it to clients
as on-demand media. This media provided from a pre-recorded source, or captured from live output, but in either case, this media is not immediately passed to viewers, but is stored for "on-demand" retrieval, and may be transcoded upon ingest.

### On-Demand Media Streaming {#od-stream}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
| **Bi-directional**| No
| **Latency**|  On Demand

Where media is received from a non-live, typically pre-recorded source. This may
feature additional outputs, bitrates, codecs, and media types described in the
live media streaming use case.

# Proposed Scope for "Media Over QUIC" {#propscope}

Our proposal is that "Media Over QUIC" discussions focus first on the use cases described in {{lm-media}}, which are Live Media Ingest ({{lmingest}}),
Syndication ({{lmsynd}}), and Streaming ({{lmstream}}). Our reasoning for this suggestion follows.

Each of the above use cases in {{overallusecases}} fit into one of three classifications of solutions.

## Analysis for Interactive Use Cases {#analy-interact}

The first group, Interactive Media, as described in {{interact}}, and covering gaming ({{gaming}}), screen sharing ({{remdesk}}), and general video conferencing ({{vidconf}}), are
largely covered by RTP, often in conjunction with WebRTC {{WebRTC}}, and related protocols today.

Whilst there may be
benefit in these use cases having a QUIC based protocol it may be more
appropriate given the size of existing deployments to extend the RTP
protocols and specifications.

## Analysis for Live Media Use Cases {#analy-lm}

The second group of classifications, in {{lm-media}}, covering Live Media Ingest ({{lmingest}}),
Live Media Syndication ({{lmsynd}}), and Live Media Streaming ({{lmstream}}) are likely the use cases that will benefit most from
this work.

Existing ingest and streaming protocols such as HLS {{RFC8216}} and DASH {{DASH}}
are reaching limits towards how low they can reduce latency in live streaming
and for scenarios where low-bitrate audio streams are used, these protocols add a significant
amount of overhead compared to the media bitstream itself.

For this reason, we suggest that work on "Media Over QUIC" protocols target these use cases at this time.

## Analysis for On-Demand Use Cases {#analy-od}

The third group, {{od-media}}, covering On-Demand Media Ingest ({{od-ingest}}) and On-Demand Media streaming ({{od-stream}}) is unlikely to benefit from work in
this space. Without the same "Live Media" latency requirements that would motivate deployment of new protocols, existing protocols such as HLS and
DASH are probably "good enough" to meet the needs of these use cases.

This does not mean that existing protocols in this space are perfect. Segmented protocols such as HLS and DASH were developed to overcome the deficiencies of TCP, as used in HTTP/1.1 {{RFC7230}} and HTTP/2 {{RFC7540}}, and do not make full use of the possible congestion window along the path from sender to receiver. Other protocols in this space have their own deficiencies. For example, RTSP {{RFC7826}} does not have easy ways to add support for new media codecs.

Our expectation is that these use cases will not drive work in the "Media Over QUIC" space, but as new protocols come into being, they may very well be taken up for these use cases as well.

# Considerations for Protocol Work {#considerations}

Even a cursory examination of the existing proposals listed in {{priorart}}
shows that there are fundamental differences in the approaches being used. This sction is intended to "up-level" the conversation beyond specific protocols, so that we can more likely agree on what is important for protocol design work.

Please note that the considerations in this section are focused especially on the use cases described in {{lm-media}}, although other use cases are mentioned for comparison and contrast.

## Here Be Dragons

The discussion in {{considerations}} is less mature than in most other sections of this document. The good news is that this section is fertile ground for people who would like to contribute to future revisions of this document. Comments are even more welcome for this section than for the rest of the document, for which they are welcome. The authors suggest that high-level comments are most appropriate at this time.

## Codec Agility

When initiating a media session, both the sender and receiver will need to agree on the codecs, bitrates, resolution, and other media details based on
capabilities and preferences. This agreement needs to take place before commencing
media transmission, but might also take place during media transmission, perhaps as a result of changes to device output or network
conditions (such as reduction in available network bandwidth).

It may be
prefered to use existing ecosystem for such purposes, e.g. SDP {{RFC4566}}.

## Support an Appropriate Range of Latencies

Support for a nominal latency appropriate for the use cases that are in scope should be
achievable, with consideration for the minimum buffer that a receiver playing content may
need to handle congestion, packet loss, and other degradation in network
quality.

## Migration of Sessions

Handling of migration of a session between hosts, either of sender or receiver
should be supported. This may either happen because the sender is undergoing
maintenence or a rebalancing of resource, because the either is experiencing a
change in network connectivity (such as a device moving from WiFi to cellular
connectivity) or other reasons.

This may depend on QUIC capabilities such as {{I-D.draft-ietf-quic-multipath}}
but support for full QUIC operation over multiple paths between senders and receivers is by no means essential.

## Appropriate Congestion Control {#acc}

An appropriate congestion control mechanism will depend upon the use cases under consideration.

It's worth remembering that we have more experience with QUIC carrying HTTP traffic than with any other type of application at this time, and consequently, we have more experience with congestion control mechanisms such as NewReno {{RFC9002}}, Cubic {{RFC8312}}, and BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}} being used with QUIC than with any other congestion control mechanisms. These congestion control mechanisms may also be appropriate for the on-demand use cases described in {{od-media}}.

Conversely, for the interactive use cases described in {{interact}}, these congestion control mechanisms are very likely inappropriate, especially when QUIC is being used with a Media Transport Protocol such as RTP, which provides its own congestion control mechanism, and which does not seem to interact well with a second, QUIC-level congestion control mechanism. Congestion control mechanisms such as SCReAM {{RFC8298}} or NADA {{RFC8698}} may be more appropriate for media. "Congestion Control Requirements for Interactive Real-Time Media" {{RFC8836}} is a useful reference.

Awkwardly, the live media use cases described in {{lm-media}} live somewhere in the middle, and work will be needed to understand the characteristics of an appropriate congestion control mechanism for these use cases.

## Support Lossless and Lossy Media Transport

TODO: confirm scope of this draft to describe lossless media transport, lossy media transport, or both lossless and lossy transport.

## Flow Directionality

Media should be able to flow in either direction from client to server or
vice-versa, either individually or concurrently but should only be negotiated at
the start of the session.

## WebTransport

TODO: Unsure of the importance of this consideration for live media use cases. If this is critical, we have to consider two
things:

- WebTransport supports HTTP/2, are we going to explicitly exclude it?
- Also, WebTransport {{I-D.draft-ietf-webtrans-overview}} has normative language
around congestion control, which may be at odds with the considerations described in {{acc}}.

## Authentication

In order to allow hosts to authenticate one another, capabilities beyond what QUIC provides may be necessary. This should be kept simple but robust in nature to
prevent attacks like credential brute-forcing.

TODO: More details are required here

## Considerations Implying QUIC Extensions

Most of the discussion of protocol work in this document has avoided mentioning capabilities that may be useful for some use cases, but seem to imply the need for extensions to the QUIC protocol, beyond what is already being considered in the IETF QUIC working group. These are included in this section, for completeness' sake.

### NAT Traversal

From Section 8.2 of {{RFC9000}}:

> Path validation is not designed as a NAT traversal mechanism. Though the mechanism described here might be effective for the creation of NAT bindings that support NAT traversal, the expectation is that one endpoint is able to receive packets without first having sent a packet on that path. Effective NAT traversal needs additional synchronization mechanisms that are not provided here.

Although there are use cases that would benefit from a mechanism for NAT traversal, a QUIC protocol extention would be needed to support those use cases.

### Multicast

Even if multicast and other network broadcasting capabilities are often used in delivering media in our use cases, QUIC doesn't yet support multicast, and a QUIC protocol extension would be needed to do so. In addition, the inclusion of multicast would introduce more complexity in both the specification and
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
