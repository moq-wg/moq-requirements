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
  I-D.draft-ietf-webtrans-overview:

  ISOBMFF:
    title: "Information Technology - Coding Of Audio-Visual Objects - Part 12: ISO Base Media File Format"
    date: 2022

  IESG-sdwg:
    target: https://www.ietf.org/about/groups/iesg/statements/support-documents/
    title: "Support Documents in IETF Working Groups"
    date: 13 Nov 2016

  MOQ-charter:
    target: https://datatracker.ietf.org/wg/moq/about/
    title: "Media Over QUIC (moq)"
    date: September 8, 2022

  WebTrans-charter:
    target: https://datatracker.ietf.org/wg/webtrans/about/
    title: "WebTransport (webtrans)"
    date: March 10, 2021

  Prog-MOQ:
    target: https://datatracker.ietf.org/meeting/interim-2022-moq-01/materials/slides-interim-2022-moq-01-sessa-moq-use-cases-and-requirements-individual-draft-working-group-draft-00
    title: "Progressing MOQ"
    date: October 21, 2022

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

This document describes use cases and requirements that guide the specification of a simple, low-latency media delivery solution for ingest and distribution {{MOQ-charter}}, using either the QUIC protocol {{RFC9000}} or WebTransport {{WebTrans-charter}} as transport protocols.

## Note for MOQ Working Group participants

When adopted, this document is intended to capture use cases that are in scope for work on the MOQ protocol {{MOQ-charter}}, and requirements that arise from these use cases.

As of this writing, the authors have not planned to request publication on this document, based on our understanding of the IESG's statement on "Support Documents in IETF Working Groups" {{IESG-sdwg}}, which says (among other things):

> While writing down such things as requirements and use cases help to get a common understanding (and often common language) between participants in the working group, support documentation doesn’t always have a high archival value. Under most circumstances, the IESG encourages the community to consider alternate mechanisms for publishing this content, such as on a working group wiki, in an informational appendix of a solution document, or simply as an expired draft.

It seems reasonable for the working group to improve this document, and then consider whether the result justifies publication as a part of the RFC archival document series.

# Terminology {#term}

{::boilerplate bcp14-tagged}

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that are in scope for "Media Over QUIC" {{MOQ-charter}}.

For each use case in this section, we also describe

* the number of senders or receiver in a given session transmitting distinct streams,
* whether a session has bi-directional flows of media from senders and receivers, which may also include timely non-media such as haptics or timed events.

It is likely that we should add other characteristics, as we come to understand them.

## Interactive Media {#interact}

The use cases described in this section have one particular attribute in common - the target the lowest possible latency as can be achieved at the trade off of data loss and complexity. For example,

* It may make sense to use FEC {{RFC6363}} and codec-level packet loss concealment {{RFC6716}}, rather than selectively retransmitting only lost packets. These mechanisms use more bytes, but do not require multiple round trips in order to recover from packet loss.
* It's generally infeasible to use congestion control schemes like BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}} in many deployments, since BBR has probing mechanisms that rely on temporarily inducing delay, but these mechanisms can then amortize the consequences of induced delay over multiple RTTs.

This may help to explain why interactive use cases have typically relied on protocols such as RTP {{RFC3550}}, which provide low-level control of packetization and transmission, with addtional support for retransmission as an optional extension.

### Gaming {#gaming}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| Yes

In this use case the computation for running a video game (single or
multiplayer) is performed externally on a hosted service, with user inputs from
input devices sent to the server, and video and audio of gameplay returned. This
may also include the client receiving other types of signaling, such as triggers
for haptic feedback, as well as media from the client such as microphone audio
for in-game chat with other players. Latency may be considerably important in
this use case as updates to video occur in response user input, with certain
genres of games having high requirements in responsiveness and/or a high
frequency of user input.

### Remote Desktop {#remdesk}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
|**Bi-directional**| Yes

Similar to the gaming use case, but where a user wishes to observe or control
the graphical user interface of another computer through local user interfaces.
Latency requirements with this use case are marginally different than the gaming
use case as greater input latency may be more tolerated by users. This use case
may also include a need to support signalling and/or transmitting of files or
devices connected to the user's computer.

### Video Conferencing/Telephony {#vidconf}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  Many to Many
|**Bi-directional**| Yes

Where media is both sent and received; This may include audio from both
microphone(s) and/or cameras, or may include "screen sharing" or inclusion of
other content such as slide, document, or video presentation. This may be done
as client/server, or peer to peer with a many to many relationship of both
senders and receivers. The target for latency may be as large as 200ms or more for
some media types such as audio, but other media types in this use case have much
more stringent latency targets.

## Hybrid Interactive and Live Media

For the video conferencing/telephony use case, there can be additional scenarios
where the audience greatly outnumbers the concurrent active participants, but
any member of the audience could participate. As this has a much larger total
number of participants - as many as Live Media Streaming {{lmstream}}, but with
the bi-directionality of conferencing, this should be considered a "hybrid". There can be additional functionality as well that overlap between the two, such as "live rewind", or recording abilities.

Another consideration is the limits of "human bandwidth" - as the number of
sources are included into a given session increase, the amount of media that can
usefully understood by a single person diminishes. To put it more simply - too
many people talking at once is much more difficult to understand than one person
speaking at a time. Subsequently this will define some limitations in the number
of concurrent or semi-concurrent, bidirectional communications that occur.

## Live Media {#lm-media}

The use cases in this section like those in {{interact}} do set some expectations to minimise high and/or highly variable latency, however their key difference is that are seldom bi-directional as their basis is on mass-consumption of media or the contribution of it into a platform to syndicate, or distribute. Latency is less noticeable over loss, and may be more accepting of having slightly more latency to increase guarantee of delivery.

### Live Media Ingest {#lmingest}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| No

Where media is received from a source for onwards handling into a distribution
platform. The media may comprise of multiple audio and/or video sources.
Bitrates may either be static or set dynamically by signaling of connection
information (bandwidth, latency) based on data sent by the receiver, and the
media may go through additional steps of transcoding or transformation before
being distributed.

### Live Media Syndication {#lmsynd}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| No

Where media is sent onwards to another platform for further distribution and not
directly used for presentation to an audience, however may be monitored by
operational systems and/or people. The media may be compressed down to a bitrate
lower than source, but larger than final distribution output. Streams may be
redundant with failover mechanisms in place.

### Live Media Streaming {#lmstream}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
|**Bi-directional**| No

Where media is received from a live broadcast or stream either as a broadcast
with fixed duration or as ongoing 24/7 output. The number of receivers may vary
depending on the type of content; breaking news events may see sharp, sudden
spikes, whereas sporting and entertainment events may see a more gradual ramp
up with a higher sustained peak with some changes based on match breaks or
interludes.

Such broadcasts may comprise of multiple audio or video outputs with different
codecs or bitrates, and may also include other types of media essence such as
subtitles or timing signalling information (e.g. markers to indicate change of
behaviour in client such as advertisement breaks). The use of "live rewind"
where a window of media behind the live edge can be made available for clients
to playback, either because the local player falls behind edge or because the
viewer wishes to play back from a point in the past.

# Requirements for Protocol Work {#req-sec}

Our goal in this section is to understand the requirements that result from the use cases described in {{overallusecases}}.

## Notes to the Reader

* Note: the intention for the requirements in this document is that they are useful for MOQ working group participants, to recognize constraints, and useful for readers outside the MOQ working group to understand the high-level functionality of the MOQ protocol, as they consider implementation and deployment of systems that rely on the MOQ protocol.

## Specific Protocol Considerations {#proto-cons}

In order to support the various topologies and patterns of media flows with the protocol, the protocol MUST support both sending and receiving of media streams, as separate actions or concurrently in a given connection.

### Delivery Assurance vs. Delay

Different use cases have varying requirements with respect to the tradeoffs associated in having guarantee of delivery vs delay - in some (such as telephony) it may be acceptable to drop some or all of the media as a result of changes in network connectivity, throughput, or congestion whereas in other scenarios all media must arrive at the receiving end even if delayed. There SHOULD be support for some means for a connection to signal which media may be abandoned, and behaviours of both senders receivers defined when delay or loss occurs. Where multiple variants of media are sent, this SHOULD be done so in a way that provides pipelining so each media stream may be processed in parallel.

### Support Webtransport/Raw QUIC as media transport

There should be a degree of decoupling from the underlying transport protocols and MoQ itself despite the "Q" in the name, in particular to provide future agility and prevent any potential ossification being tied to specific version(s) of dependant protocols.

Many of the use cases will be deployed in contexts where web browsers are the common application runtime; thus the use of existing protocols and APIs is desireable for implementations. Support for WebTransport {{I-D.draft-ietf-webtrans-overview}} will be defined, although implementations or deployments running outside browsers will not need to use WebTransport, thus support for the protocol running directly atop QUIC should be provided.

Considerations should be made clear with respect to modes where WebTransport "falls back" to using HTTP/2 or other future non-QUIC based protocol.

### Media Negotiation & Agility {#MOQ-negotiation}

All entities which directly process media will have support for a variety of media codecs, both codecs which exist now and codecs that will be defined in the future. Consequently the protocol will provide the capability for sender and receiver to negotiate which media codecs will be used in a given session.

The protocol SHOULD remain codec agnostic as much as possible, and should allow for new media formats and codecs to be supported without change in specification.

The working group should consider if a minimum, suggestive set of codecs should be supported for the purposes of interop, however this SHOULD avoid being strict to simplify use cases and deployments that don't require certain capability e.g. telephony which may not require video codecs.

## Media Data Model

As the protocol will handle many different types of media, classifications, and variations when all entities describe the media a model should be defined which represents this, with a clear addressing scheme. This should factor in at least, but not limited to allow future types:

Media Types
: Video, audio, subtitles, ancillary data

Classifications
: Codec, language, layers

Variations
: For each stream, the resolution(s), bitrate(s). Each variant should be uniquely identifiable and addressable.

Considerations should be made to addressing of individual audio/video frames as opposed to groups, in addition to how the model incorporates signalling of prioritisation, media dependency, and cacheability to all entities.

## Publishing Media {#pub-media}

Many of the use cases have bi-directional flows of media, with clients both sending and receiving media concurrently, thus the protocol should have a unified approach in connection negotiation and signalling to send and received media both at the start and ongoing in the lifetime of a session including describing when flow of media is unsupported (e.g. a live media server signalling it does not support receiving from a given client).

In the initiation of a session both client and server must perform negotiation in order to agree upon a variety of details before media can move in any direction:

* Is the client authenticated and subsequently authorised to initiate a connection?
* What media is available, and for each what are the parameters such as codec, bitrate, and resolution etc?
* Can media move bi-directionally, or is it unidirectional only?

## Naming and Addressing Media Resources {#naming}

As multiple streams of media may be available for concurrent sending such as multiple camera views or audio tracks, a means of both identifying the technical properties of each resource (codec, bitrate, etc) as well as a useful identification for playback should be part of the protocol. A base level of optional metadata e.g. the known language of an audio track or name of participant's camera should be supported, but further extended metadata of the contents of the media or its ontology should not be supported.

### Scoped to an Origin/Domain, Application specific.

### Allows subscribing or requesting for the data matching the name by the consumers

## Packaging Media {#Packaging}

Packaging of media describes how raw media will be encapsulated. There are at a high level two approaches to this:

* Within the protocol itself, where the protocol defines the ancillary data required to decode each media type the protocol supports.
* A common encapsulation format such as ISOBMFF {{ISOBMFF}} which defines a generic method for all media and handles ancillary decode information.

The working group must agree on which approach should be taken to the packaging of media, taking into consideration the various technical trade offs that each approach provides.

- If the working group decides to describe media encapsulation as part of the MOQ protocol, this will require a new version of the MOQ protocol in order to signal the receiver that a new media encapsulation format may be present.

- If the working group decides to use a common encapsulation format, the mechanisms within the protocol SHOULD allow for new encapsulation formats to be used. Without encapsulation agility, adding or changing the way media is encapsulated will also require a new version of the MOQ protocol, to signal the receiver that a new media encapsulation format may be present.

MOQ protocol specifications will provide details on the supported media encapsulation(s).

## Media Consumption {#med-consumption}

Receivers SHOULD be able to as part of negotiation of a session {{MOQ-negotiation}} specify which media to receive, not just with respect to the media format and codec, but also the varient thereof such as resolution or bitrate.

## Relays, Caches, and other MOQ Network Elements {#MOQ-network-entities}

### Pull & Push

To enable use cases where receivers may wish to address a particular time of media in addition to having the most recently produced media available, both "pull" and "push" of media SHOULD be supported, with consideration that producers and intermediates SHOULD also signal what media is available (commonly referred to as a "DVR window"). Behaviours around cache durations for each MoQ entity should be defined.

## Security {#MOQ-security}

### Authentication & Authorisation

Whilst QUIC and conversely TLS supports the ability for mutual authentication through client and server presenting certificates and performing validation, this is infeasible in many use cases where provisioning of client TLS certificates is unsupported or infeasible. Thus, support for a primitive method of authentication between MoQ entities SHOULD be included to authenticate entities between one another, noting that implementations and deployments should determine which authorisation model if any is applicable.

### Media Encryption {#MOQ-media-encryption}

End-to-end security describes the use of encryption of the media stream(s) to provide confidentiality in the presence of unauthorized intermediates or observers and prevent or restrict ability to decrypt the media without authorization. Generally, there are three aspects of end-to-end media security:

* Digital Rights Management, which refers to the authorization of receivers to decode a media stream.
* Sender-to-Receiver Media Security, which refers to the ability of media senders and receivers to transfer media while protected from authorized intermediates and observers, and
* Node-to-node Media Security, which refers to security when authorized intermediaries are needed to transform media into a form acceptable to authorized receivers. For example, this might refer to a video transcoder between the media sender and receiver.

**Note: "Node-to-node" refers to a path segment connecting two MOQ nodes, that makes up part of the end-to-end path between the MOQ sender and ultimate MOQ receiver.

Support for encrypted media SHOULD be available in the protocol to support the above use cases, with key exchange and decryption authorisation handled externally. The protocol SHOULD provide metadata for entities which process media to perform key exchange and decrypt.

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
