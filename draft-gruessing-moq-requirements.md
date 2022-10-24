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
  I-D.draft-jennings-moq-quicr-arch:
  I-D.draft-jennings-moq-quicr-proto:

  MOQ-charter:
    target: https://datatracker.ietf.org/wg/moq/about/
    title: "Media Over QUIC (moq)"
    date: September 8, 2022
  Prog-MOQ:
    target: https://datatracker.ietf.org/meeting/interim-2022-moq-01/materials/slides-interim-2022-moq-01-sessa-moq-use-cases-and-requirements-individual-draft-working-group-draft-00
    title: "Progressing MOQ"
    date: October 21, 2022
  MOQ-ucr:
    target: https://datatracker.ietf.org/meeting/interim-2022-moq-01/materials/slides-interim-2022-moq-01-sessa-progressing-moq-00.pdf
    title: "MOQ Use Cases and Requirements"
    date: October 21, 2022
  DASH:
    target: https://www.iso.org/standard/79329.html
    title: "ISO/IEC 23009-1:2019: Dynamic adaptive streaming over HTTP (DASH) -- Part 1: Media presentation description and segment formats (2nd edition)"
  WebRTC:
    target: https://www.w3.org/groups/wg/webrtc
    title: "Web Real-Time Communications Working Group"

--- abstract

This document describes the chartered use cases that guide development of a simple, low-latency media delivery solution for ingest and distribution of media, and the requirements for this solution that result from these use cases.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MoQ)
mailing list, at <https://www.ietf.org/mailman/listinfo/moq>.

--- middle

# Introduction {#intro}

This document describes the chartered {{MOQ-charter}} use cases that guide development of a simple, low-latency media delivery solution for ingest and distribution of media, and the requirements for this solution that result from these use cases.

## Intention for this version of the document

RFC Editor: If this document reaches you, please remove this section!

This version of the document is intended to provide the MOQ working group with a starting point for work on the "Use Cases and Requirements document" milestone, as described in {{MOQ-ucr}}.

# Terminology {#term}

(To Do)

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that are in scope for "Media Over QUIC" {{MOQ-charter}}.

For each use case in this section, we also describe

* the number of senders or receiver in a given session transmitting distinct streams,
* whether a session has bi-direction flows of media from senders and receivers, and
* the expected lowest latency requirements.

It is likely that we should add other characteristics, as we come to understand them.

## Interactive Media {#interact}

The use cases described in this section have one particular attribute in common - the target latency for these cases are on the order of one or two RTTs. In order to meet those targets, it is not possible to rely on protocol mechanisms that require multiple RTTs to function effectively. For example,

* When the target latency is on the order of one RTT, it makes sense to use FEC {{RFC6363}} and codec-level packet loss concealment {{RFC6716}}, rather than selectively retransmitting only lost packets. These mechanisms use more bytes, but do not require multiple RTTs in order to recover from packet loss.
* When the target latency is on the order of one RTT, it is impossible to use congestion control schemes like BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}}, since BBR has probing mechanisms that rely on temporarily inducing delay and amortizing the consequences of that over multiple RTTs.

This may help to explain why interactive use cases have often relied on protocols such as RTP {{RFC3550}}, which provide low-level control of packetization and transmission, and make no provision for retransmission.

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

# Requirements for Protocol Work {#req-sec}

*Note: the initial high-level organization for this section is taken from Suhas Nandakumar's presentation, "Progressing MOQ" {{Prog-MOQ}}, at the October 2022 MOQ virtual interim meeting, which was in turn taken from the MOQ working group charter {{MOQ-charter}}. We'll move this mention to the acknowledgement section, if the structure changes significantly.*

## Common Publication Protocol Between Ingest and Distribution {#pub-proto}

## Naming and Addressing Media {#naming}

## Packaging Media {#Packaging}

## Client Media Request Protocol {#media-request}

## End to End Security {MOQ-security}

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to guide discussion and consensus, it introduces
no security considerations of its own.

--- back

# Acknowledgements

The authors would like to thank several authors of individual drafts that provided input during the "Media Over QUIC" charter process:

- Kirill Pugin, Alan Frindell, Jordi Cenzano, and Jake Weissman ({{I-D.draft-kpugin-rush}},
- Luke Curley ({{I-D.draft-lcurley-warp}}), and
- Cullen Jennings and Suhas Nandakumar ({{I-D.draft-jennings-moq-quicr-arch}}), together with Christian Huitema ({{I-D.draft-jennings-moq-quicr-proto}}).

James Gruessing would also like to thank Francesco Illy and Nicholas Book for
their part in providing the needed motivation.
