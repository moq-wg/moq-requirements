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
  WebTrans-charter:
    target: https://datatracker.ietf.org/wg/webtrans/about/
    title: "WebTransport (webtrans)"
    date: March 10, 2021

--- abstract

This document describes use cases and requirements that guide the specification of a simple, low-latency media delivery solution for ingest and distribution, using either the QUIC protocol or WebTransport.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

Discussion of this draft should take place on the IETF Media Over QUIC (MoQ)
mailing list, at <https://www.ietf.org/mailman/listinfo/moq>.

--- middle

# Introduction {#intro}

This document describes use cases and requirements that guide the specification of a simple, low-latency media delivery solution for ingest and distribution {{MOQ-charter}}, using either the QUIC protocol {{RFC9000}} or WebTransport {{WebTrans-charter}}.

## Note for MOQ Working Group participants

This version of the document is intended to provide the MOQ working group with a starting point for work on the "Use Cases and Requirements document" milestone. The update implements the work plan described in {{MOQ-ucr}}. The authors intend to request MOQ working group adoption after IETF 115, so the working group can begin to focus on these topics in earnest.

# Terminology {#term}

(To Do)

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that are in scope for "Media Over QUIC" {{MOQ-charter}}.

For each use case in this section, we also describe

* the number of senders or receiver in a given session transmitting distinct streams,
* whether a session has bi-directional flows of media from senders and receivers, and
* the worst-case expected RTT requirements.

It is likely that we should add other characteristics, as we come to understand them.

## Interactive Media {#interact}

The use cases described in this section have one particular attribute in common - the target latency for these cases are on the order of one or two RTTs. In order to meet those targets, it is not possible to rely on protocol mechanisms that require multiple RTTs to function effectively. For example,

* When the target latency is on the order of one RTT, it makes sense to use FEC {{RFC6363}} and codec-level packet loss concealment {{RFC6716}}, rather than selectively retransmitting only lost packets. These mechanisms use more bytes, but do not require multiple RTTs in order to recover from packet loss.
* When the target latency is on the order of one RTT, it is impossible to use congestion control schemes like BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}}, since BBR has probing mechanisms that rely on temporarily inducing delay, but these mechanisms can then amortize the consequences of induced delay over multiple RTTs.

This may help to explain why interactive use cases have typically relied on protocols such as RTP {{RFC3550}}, which provide low-level control of packetization and transmission, and make no provision for retransmission.

### Gaming {#gaming}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| Yes
| **Latency**|  Ull-50

Where media is received, and user inputs are sent by the client. This may also
include the client receiving other types of signaling, such as triggers for
haptic feedback. This may also carry media from the client such as microphone
audio for in-game chat with other players.

### Remote Desktop {#remdesk}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
| **Bi-directional**| Yes
| **Latency**|  Ull-50

Where media is received, and user inputs are sent by the client. Latency
requirements with this use case are marginally different than the gaming use
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
the bi-directionality of conferencing, this should be considered a "hybrid".

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
Bitrates may either be static or set dynamically by signaling of connection
information (bandwidth, latency) based on data sent by the receiver.

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

Our goal in this section is to understand the requirements that result from the use cases described in {{overallusecases}}.

*Note: the initial high-level organization for this section is taken from Suhas Nandakumar's presentation, "Progressing MOQ" {{Prog-MOQ}}, at the October 2022 MOQ virtual interim meeting, which was in turn taken from the MOQ working group charter {{MOQ-charter}}. We think this is a reasonable starting point. We won't be surprised to see the high-level structure change a bit as things develop, but we didn't want to have this section COMPLETELY blank when we request working group adoption.

TODO: Describe overall, high level requirements that we previously stated in earlier versions of this document.

## Common Publication Protocol for Media Ingest and Distribution {#pub-proto}

Many of the use cases have bi-directional flows of media, with clients both sending and receiving media concurrently, thus the protocol should have a unified approach in connection negotiation and signalling to send and received media both at the start and ongoing in the lifetime of a session including describing when flow of media is unsupported (e.g. a live media server signalling it does not support receiving from a given client).

## Client Media Request Protocol {#media-request}

In the initiation of a session both client and server must perform negotiation in order to agree upon a variety of details before media can move in any direction:

* Is the client authenticated and subsequently authorised to initiate a connection?
* What media is available, and for each what are the parameters such as codec, bitrate, and resolution etc?
* Is sending of media from a client permitted? If so, what media is accepted?

Re-negotiation in an existing protocol should be supported to allow changes in what is being sent of received.

## Naming and Addressing Media Resources {#naming}

As multiple streams of media may be available for concurrent sending such as multiple camera views or audio tracks, a means of both identifying the technical properties of each resource (codec, bitrate, etc) as well as a useful identification for playback should be part of the protocol. A base level of optional metadata e.g. the known language of an audio track or name of participant's camera should be supported, but further extended metadata of the contents of the media or its ontology should not be supported.

## Packaging Media {#Packaging}

Packaging of media describes how encapsulation of media to carry the raw media will work. There are at a high level two approaches to this:

* Within the protocol itself, where the protocol defines the carrying for each media encoding the ancillary data required for decoding the media.
* A common encapsulation format such as ISOBMFF which defines a generic method for all media and handles ancillary decode information.

The working group must agree on which approach should be taken to the packaging of media, taking into consideration the various technical trade offs that each provide.

## End-to-end Security {#MOQ-security}

End-to-end security describes the use of encryption of the media stream(s) to provide confidentiality in the presence of intermediates or receiver's and prevent or restrict ability to decrypt the media. There are two primary use cases for this:

* Media Confidentiality, which applies primarily to video conferencing and telephony use cases where participants in a call want to assert that an intermediate is unable to observe their private conversation.
* Media Rights Management, which applies to live media use cases where controls on receivers are authorised to decode a media stream.

Both media confidentiality and rights management use cases should be supported in the signalling of the protocol in a unified way to inform intermediates and receivers that the media stream is encrypted, including an identifier to describe what system is used to encrypt the media, as well as any other opaque information carried to aid in decryption. Further this requires the packaging of media to support carrying encrypted media payloads. Key negotiation and exchange should happen externally of the core protocol, however consideration must be made to support signaling of changes to keying in an existing connection to support re-keying without re-negotiating the session.

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to guide discussion and consensus, it introduces
no security considerations of its own.

--- back

# Acknowledgements

The authors would like to thank several authors of individual drafts that fed into the "Media Over QUIC" charter process:

- Kirill Pugin, Alan Frindell, Jordi Cenzano, and Jake Weissman ({{I-D.draft-kpugin-rush}},
- Luke Curley ({{I-D.draft-lcurley-warp}}), and
- Cullen Jennings and Suhas Nandakumar ({{I-D.draft-jennings-moq-quicr-arch}}), together with Christian Huitema ({{I-D.draft-jennings-moq-quicr-proto}}).

We would also like to thank Suhas Nandakumar for his presentation, "Progressing MOQ" {{Prog-MOQ}}, at the October 2022 MOQ virtual interim meeting. We used his outline as a starting point for the Requirements section ({{req-sec}}).

James Gruessing would also like to thank Francesco Illy and Nicholas Book for
their part in providing the needed motivation.
