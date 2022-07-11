---
title: Media Over QUIC - Use Cases and Requirements for Media Transport Protocol Design
abbrev: MoQ Use Cases and Requirements
docname: draft-gruessing-moq-requirements-latest
category: info
stream: IETF
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
  RFC6363:
  RFC6716:
  RFC7230:
  RFC7540:
  RFC7826:
  RFC8216:
  RFC9000:

  I-D.draft-cardwell-iccrg-bbr-congestion-control:
  I-D.draft-kpugin-rush:
  I-D.draft-lcurley-warp:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-jennings-moq-quicr-arch:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-ietf-mops-streaming-opcons:
  I-D.draft-ietf-quic-datagram:
  I-D.draft-ietf-quic-http:
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
  WebRTC:
    target: https://www.w3.org/groups/wg/webrtc
    title: "Web Real-Time Communications Working Group"
  QUIC-goals:
    target: https://datatracker.ietf.org/doc/charter-ietf-quic/01/
    title: "Initial Charter for QUIC Working Group"
    date: October 4, 2016

--- abstract

This document describes use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", provides analysis about those use cases, recommends a subset of use cases that cover live media ingest, syndication, and streaming for further exploration, and describes requirements that should guide the design of protocols to satisfy these use cases.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MoQ)
mailing list, at <https://www.ietf.org/mailman/listinfo/moq>.

--- middle

# Introduction {#intro}

This document describes use cases that have been discussed in the IETF community under the banner of "Media Over QUIC", provides analysis about those use cases, recommends a subset of use cases that cover live media ingest, syndication, and streaming for further exploration, and describes requirements that should guide the design of protocols to satisfy these use cases.

## For The Impatient Reader

- Our proposal is to focus on live media use cases, as described in {{propscope}}, rather than on interactive media use cases or on-demand use cases.
- The reasoning behind this proposal can be found in {{analy-interact}}.
- The requirements for protocol work to satisfy the proposed use cases can be found in {{req-sec}}.

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
- the encapsulation of any Media Transport Protocol in a QUIC payload
- an IETF mailing list ({{MOQ-ml}}), which was requested "... for discussion of video ingest and distribution protocols that use QUIC as the underlying transport", although discussion of other Media Over QUIC proposals have also been discussed there.

There may be IETF participants using other meanings as well.

As of this writing, the second bullet ("any kind of media carried indirectly over the QUIC protocol, as an RTP payload"), seems to be in scope for the IETF AVTCORE working group, and was discussed at some length at the February 2022 AVTCORE working group meeting {{AVTCORE-2022-02}}, although no drafts in this space have yet been adopted by the AVTCORE working group.

## Latency Requirement Categories {#latent-cat}

}The "Operational Considerations for Streaming Media" document ({{I-D.draft-ietf-mops-streaming-opcons}}) described a range of latencies of interest to streaming media providers, as

- ultra low-latency (less than 1 second)
- low-latency live (less than 10 seconds)
- non-low-latency live (10 seconds to a few minutes)
- on-demand (hours or more)

Because the IETF Media Over QUIC community now expresses interest in interactive media {{interact}} and live media {{lm-media}}} use cases will have requirements that are significantly less than the "streaming media"-defined "ultra-low latency".

Within this document, we are using

- near real-time (less than 50 ms)
- Ull-200 (less than 200 ms)

Perhaps obviously, these last two latency bands are the shortened form of "ultra-low latency - 50 ms" and "ultra-low-latency - 200 ms".

Perhaps less obviously, bikeshedding on better names and more useful values is welcomed.

# Prior and Existing Specifications {#priorart}

Several draft specifications have been proposed which either encapsulate
existing Media Transport Protocols in QUIC
({{I-D.draft-sharabayko-srt-over-quic}}), make use of RTP, RTCP, and SDP
({{I-D.draft-engelbart-rtp-over-quic}}) or define their own new Media Transport
Protocol on top of QUIC. Some have already seen deployment into the wild (e.g.
{{I-D.draft-kpugin-rush}}, {{I-D.draft-lcurley-warp}}) where as others are
unconfirmed. Whilst most just focus on defining wire format,
{{I-D.draft-jennings-moq-quicr-arch}} defines an architecture using a pub/sub
model for both producers and consumers.

## Comparison of Existing Specifications

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

# Requirements for Protocol Work {#req-sec}

TODO: Quite a lot, really ...

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to guide discussion and consensus, it introduces
no security considerations of its own.

--- back

# Acknowledgements

The authors would like to thank the many authors of the specifications
referenced in {{priorart}} for their work. The authors would also like to thank
Alan Frindell, Luke Curley, and Maxim Sharabayko for text contributions to this
draft.

James Gruessing would also like to thank Francesco Illy and Nicholas Book for
their part in providing the needed motivation.
