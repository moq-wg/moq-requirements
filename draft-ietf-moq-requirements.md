---
title: Media Over QUIC - Use Cases and Requirements for Media Transport Protocol Design
abbrev: MoQ Use Cases and Requirements
docname: draft-ietf-moq-requirements-latest
category: info
stream: IETF
date:

ipr: trust200902
area: applications
workgroup: Media Over QUIC
keyword: Internet-Draft QUIC
stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, subcompact, comments, inline]

venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  github: moq-wg/moq-requirements
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  latest: "https://moq-wg.github.io/moq-requirements/draft-ietf-moq-requirements.html"

author:
  -
    ins: J. Gruessing
    name: James Gruessing
    organization: Nederlandse Publieke Omroep
    email: james.ietf@gmail.com
    country: Netherlands
  -
    ins: S. Dawkins
    name: Spencer Dawkins
    organization: Tencent America LLC
    email: spencerdawkins.ietf@gmail.com
    country: United States

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

  CMAF:
    title: "Information technology — Multimedia application format (MPEG-A) — Part 19: Common media application format (CMAF) for segmented media"
    date: 2020

  ISOBMFF:
    title: "Information Technology - Coding Of Audio-Visual Objects - Part 12: ISO Base Media File Format"
    date: 2022

  SCTE-35:
    target: https://www.scte.org/standards/library/catalog/scte-35-digital-program-insertion-cueing-message/
    title: "Digital Program Insertion Cueing Message (SCTE-35)"
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

This document describes use cases and requirements that guide the specification of a simple, low-latency media delivery solution for ingest and distribution, using either the QUIC protocol or WebTransport as transport protocols.

--- middle

# Introduction {#intro}

This document describes use cases and requirements that guide the specification of a simple, low-latency media delivery solution for ingest and distribution {{MOQ-charter}}, using either the QUIC protocol {{RFC9000}} or WebTransport {{WebTrans-charter}} as transport protocols.

## Note for MOQ Working Group Participants

When adopted, this document is intended to capture use cases that are in scope for work on the MOQ protocol {{MOQ-charter}}, and requirements that arise from these use cases.

As of this writing, the authors have not planned to request publication on this document, based on our understanding of the IESG's statement on "Support Documents in IETF Working Groups" {{IESG-sdwg}}, which says (among other things):

> While writing down such things as requirements and use cases help to get a common understanding (and often common language) between participants in the working group, support documentation doesn’t always have a high archival value. Under most circumstances, the IESG encourages the community to consider alternate mechanisms for publishing this content, such as on a working group wiki, in an informational appendix of a solution document, or simply as an expired draft.

It seems reasonable for the working group to improve this document, and then consider whether the result justifies publication as a part of the RFC archival document series.

# Terminology {#term}

{::boilerplate bcp14-tagged}

## Distinguishing between Interactive and Live Streaming Use Cases

The MOQ charter {{MOQ-charter}} lists three use cases as being in scope of the MOQ protocol

> use cases including live streaming, gaming, and media conferencing

but does not include (directly or by reference) a definition of "live streaming" or "interactive" (a term that has been used to describe gaming and media conferencing, as distinct from "live streaming"). It seems useful to describe these two terms, as classes of use cases, before we describe individual use cases in more detail.

MOQ participants have discussed making this distinction based on quantitative measures such as latency, but since MOQ use cases can include an arbitrary number of relays, we offer a distinction that is based on how users experience that distinction. If two users are able to interact in the way that seems interactive, as described in the proposed definitions, the use case is interactive; if two users are unable to interact in that way, the use case is live streaming.

We propose these definitions:

**Interactive**:
: a use case with coupled bidirectional media flows

Interactive use cases have bidirectional media flows sufficiently coupled with each other, that media from one sender can cause the receiver to reply by sending its own media back to the original sender.

For instance, a speaker in a conferencing application might make a statement, and then ask, "but what do you folks think?" If one of the listeners is able to answer in a timeframe that seems natural, without waiting for the current speaker to explicitly "hand over" control of the conversation, this would qualify as "Interactive".

**Live Streaming**:
: a use case with unidirectional media flows, or uncoupled bidirectional flows

Live Streaming use cases allow consumers of media to "watch together", without having a sense that one consumer is experiencing the media before another consumer. This does not require the delivery of live media to be strictly synchronized between media consumers, but only that from the viewpoint of individual consumers, media delivery **appears to be** synchronized.

It is common for live streaming use cases to send media in one direction, and "something else" in the other direction - for instance, a video receiver might be returning requests that the sender change the media encoding or media rate in use, or reorient a camera. This type of feedback doesn't qualify as "bidirectional media".

If two sender/receivers are each sending media to the other, but what's being carried in one direction has no relationship with what's being carried in the other direction, this would not qualify as "Interactive".

**Note: these descriptions are a starting point. Feedback and push-back are both welcomed.**

## Alignment with Terminology in Related Drafts {#draft-alignment}

The MOQ working group has used a wide variety of terms, with some, but not enough, effort to reconcile terms in use in various drafts. As these drafts are being adopted by the MOQ working group, it seems right to make every effort to align terminology for ease of reading and implementation.

This draft does not yet, but will, align with the terminology defined in {{!I-D.draft-ietf-moq-transport}}, as a starting point. We note that -00 of that draft observes that the working group hasn't converged on terminology and definitions, and if MOQ terminology changes, the terminology in this draft will change accordingly.

# Use Cases Informing This Proposal {#overallusecases}

Our goal in this section is to understand the range of use cases that are in scope for "Media Over QUIC" {{MOQ-charter}}.

For each use case in this section, we also describe

- the number of senders or receivers in a given session transmitting distinct streams,
- whether a session has bi-directional flows of media from senders and receivers, which may also include timely non-media such as haptics or timed events.

It is likely that we should add other characteristics, as we come to understand them.

## Interactive Media {#interact}

The use cases described in this section have one particular attribute in common - they target the lowest possible latency as can be achieved at the trade off of data loss and complexity. For example,

- It may make sense to use FEC {{RFC6363}} and codec-level packet loss concealment {{RFC6716}}, rather than selectively retransmitting only lost packets. These mechanisms use more bytes, but do not require multiple round trips in order to recover from packet loss.
- It's generally infeasible to use congestion control schemes like BBR {{I-D.draft-cardwell-iccrg-bbr-congestion-control}} in many deployments, since BBR has probing mechanisms that rely on temporarily inducing delay, but these mechanisms can then amortize the consequences of induced delay over multiple RTTs.

This may help to explain why interactive use cases have typically relied on protocols such as RTP {{RFC3550}}, which provide low-level control of packetization and transmission, with additional support for retransmission as an optional extension.

To provide an overview of interactive use cases, we can consider a conferencing session comprised of:

- Multiple emitters, publishing on multiple tracks (audio, video tracks and at different qualities)

- A media switch, sourcing tracks that represent a subset of tracks from across all the emitters. For example, such a subset may represent tracks representing top 5 speakers at higher qualities and lot of other tracks for rest of the emitters at lower qualities.

- Multiple receivers, with varied receiving capacity (bandwidth limited), subscribing to subset of the tracks


~~~
                                   SFU:t1, E1:t2, E3:t6
 .───.  E1: t1,t2,t3,t4                          .───.
( E1  )─────┐                           ┌────▶ ( R1  )
 `───'      │                           │       `───'
            │                           │
            └───────▶─────────┐         │
                     │         │────────┘
 .───.  E2: t1,t2    │   SFU   │   SFU:t1,E1:t2 .───.
( E2  )─────────────▶│         │──────────────▶( R2  )
 `───'               │         │                `───'
           ┌────────▶└─────────┴─────────┐
           │                             │
           │                             │
           │                             │
           │                             │
 .───.     │                             │       .───.
( E3  )────┘                             └─────▶( R3  )
 `───'   E3: t1,t2,t3,t4,t5,t6          E3: t2,  `───'
                                        E1: t2,
                                        E2: t2,
                                        SFU: t1
~~~



This setup relies on the following functionalities:

- Media Switches source new tracks but retain the media payload from the original emitters. This implies publishing new Track IDs sourced from the SFU, with object payload unchanged from the original emitters.

- Media Switches propagate a subset of tracks as-is from the emitters to the subscribers. This implies Track IDs to be unchanged between the emitters and the receivers.

- Subscribers explicitly request one or more media tracks in appropriate qualities and dynamically move between the qualities during the course of the session.

Another topology for the conferencing use-case is to use multiple distribution networks for delivering the media, with media switching functionality running across distribution networks and these media functions as part of the core distribution network as shown below.

~~~
                   Distribution Network A
 E1: t1,t2,t3,t4
                                    SFU:t1, E1:t2, E3:t6
    .───.        ┌────────┐      ┌────────┐      .───.
   ( E1  )───────│ Relay  │──────│ Relay  ├───▶ ( R1  )
    `───'        └─────┬──┘      └──┬─────┘      `───'
                       │ ┌────────┐ │
   E2: t1,t2           └─┤ Relay  │─┘
             ┌──────────▶└────┬───┘         SFU:t1,E1:t2
    .───.    │                 │                  .───.
   ( E2  )───┘                 │              ┌─▶( R2  )
    `───'                      │              │   `───'
                   ┌────────┐  │   ┌────────┬─┘
             ──────┤ Relay  │──┴───│ Relay  │─┐
             |     └─────┬──┘      └──┬─────┘ │
             |           │ ┌────────┐ │       │
             |           └─┤ Relay  │─┘       │
    .───.    |             └────────┘         │   .───.
   ( E3  )───┘         Distribution Network B └─▶( R3  )
    `───'                                         `───'
     E3: t1,t2,t3,t4,t5,t6                        E3: t2,
                                                  E1: t2,
                                                  E2: t2,
                                                 SFU: t1
~~~

Such a topology needs to meet all the properties listed in the homogenous topology setup, however having multiple distribution networks, and relying on the distribution networks to carry out the media delivery, brings in further requirements towards a data model that enables tracks to be uniquely identifiable across the distribution networks and not just within a single distribution network.

### Gaming {#gaming}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| Yes

In this use case the computation for running a video game (single or multiplayer) is performed externally on a hosted service, with user inputs from input devices sent to the server, and media, usually video and audio of gameplay returned. This may also include the client receiving other types of signaling, such as triggers for haptic feedback, as well as the client sending media such as microphone audio for in-game chat with other players. Latency may be considerably important in this use case as updates to video occur in response user input, with certain genres of games having high requirements in responsiveness and/or a high frequency of user input.

### Remote Desktop {#remdesk}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
|**Bi-directional**| Yes

Similar to the gaming use case in many requirements, but where a user wishes to observe or control the graphical user interface of another computer through local user interfaces. Latency requirements with this use case are marginally different than the gaming use case as greater input latency may be more tolerated by users. This use case may also include a need to support signalling and/or transmitting of files or devices connected to the user's computer.

### Video Conferencing/Telephony {#vidconf}

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  Many to Many
|**Bi-directional**| Yes

In the Video Conferencing/Telephony use case, media is both sent and received. This use case typically includes audio and video media, and may also include one or more  additional media types, such as "screen sharing" and other content such as slides, documents, or video presentations. This may be done as client/server, or peer to peer with a many to many relationship of both senders and receivers. The target for latency may be as large as 200ms or more for some media types such as audio, but other media types in this use case have much more stringent latency targets.

## Live Media {#lm-media}

The use cases in this section like those in {{interact}} do set some expectations to minimize high and/or highly variable latency, however their key difference is that are seldom bi-directional as their basis is on mass-consumption of media or the contribution of it into a platform to syndicate, or distribute. Latency is less noticeable over loss, and may be more accepting of having slightly more latency to increase guarantee of delivery.

### Live Media Ingest {#lmingest}

In a typical live video ingest, the broadcast client - for example, an Open Broadcaster Software (OBS) client, publishes the video content to an ingest server under a provider domain.

~~~

               E1: t1,t2,t3   ┌──────────┐
 .─────────────.              │          │
(    Emitter    )────────────▶│  Ingest  │
 `─────────────'              │  Server  │
                              │          │
                              └──────────┘

~~~

The Track IDs are scoped to the broadcast for the application under a provider domain.

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| No

Where media is received from a source for onwards handling into a distribution platform. The media may comprise of multiple audio and/or video sources. Bitrates may either be static or set dynamically by signaling of connection information (bandwidth, latency) based on data sent by the receiver, and the media may go through additional steps of transcoding or transformation before being distributed.

### Live Media Syndication {#lmsynd}

**Note: We need to add a description for Live Media Syndication, matching the descriptions of {{lmingest}} and {{lmstream}}**

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to One
|**Bi-directional**| No

Where media is sent onwards to another platform for further distribution and not directly used for presentation to an audience, however may be monitored by operational systems and/or people. The media may be compressed down to a bitrate lower than source, but larger than final distribution output. Streams may be redundant with failover mechanisms in place.

### Live Media Streaming {#lmstream}

In a reference live streaming example shown below, the emitter streams one or more tracks as part of the application operated under a provider domain, which is then distributed to multiple clients using some form of distribution server operating under the same provider domain, over a content distribution network.

~~~

                                                                 DS: t1,t2
                                                                   .───.
                                                          ┌──────▶( S1  )
                                                          │        `───'
                                                          │
        E1: t1,t2,t3 ┌──────────┐    ┌──────────────┬─────┘     DS: t1
.─────────.          │          │    │              │         .───.
(   E1    )─────────▶│  Ingest  ├────┤ Distribution │───────▶( S2  )
`─────────'          │  Server  │    │      Server  |         `───'
                     │          │    │              │
                     └──────────┘    └──────────────┴─────┐
                                                          │        .───.
                                                          └──────▶( S3  )
                                                                   `───'
                                                                DS: t1,t2, t3
~~~

In this setup, one can visualize the ingest and distribution as two separate systems operating within a given provider domain. One implication of this organization is that the Track Ids used by the emitter need not match the ones referred to by the subscribers. This can be the case because the distribution server sources the media as new tracks (for instance, if the media is transcoded after ingest)

**Note: the previous paragraph describes the relationship between Live Media Ingest and Live Media Streaming - it might better go in a separate subsection, and should also include Live Media Syndication**

|Attribute | Value |
| -----+----------------
|**Senders/Receivers**|  One to Many
|**Bi-directional**| No

In Live Media Streaming, media might be received from a live broadcast or stream either as a broadcast with fixed duration or as ongoing 24/7 output. The number of receivers may vary depending on the type of content; breaking news events may see sharp, sudden spikes, whereas sporting and entertainment events may see a more gradual ramp up with a higher sustained peak with some changes based on match breaks or interludes.

Such broadcasts may comprise of multiple audio or video outputs with different codecs or bitrates, and may also include other types of media essence such as subtitles or timing signalling information (e.g. markers to indicate change of behavior in client such as advertisement breaks). The use of "live rewind" where a window of media between the live edge and trailing edge can be made available for clients to playback, either because the local player falls behind the leading edge or because the viewer wishes to play back from a point in the past.

## Hybrid Interactive and Live Media

For the video conferencing/telephony use case, there can be additional scenarios where the audience greatly outnumbers the concurrent active participants, but any member of the audience could participate. This use case can have an audience as large as Live Media Streaming as described in {{lmstream}}, but also relies on the interactivity and bi-directionality of conferencing as in Video Conferencing as described in {{vidconf}}. For this reason, this type of use case can be considered a "hybrid". There can be additional functionality as well that overlap between the two, such as "live rewind", or recording abilities.

Another consideration is the limits of "human bandwidth" - as the number of sources are included into a given session increase, the amount of media that can usefully understood by a single person diminishes. To put it more simply - too many people talking at once is much more difficult to understand than one person speaking at a time, and this varies on the audience and circumstance. Subsequently this will define some limitations in the number of potential concurrent or semi-concurrent, bidirectional communications that occur.

# Requirements for Protocol Work {#req-sec}

Our goal in this section is to understand the requirements that result from the use cases described in {{overallusecases}}.

## Notes to the Reader

* Note: the intention for the requirements in this document is that they are useful for MOQ working group participants, to recognize constraints, and useful for readers outside the MOQ working group to understand the high-level functionality of the MOQ protocol, as they consider implementation and deployment of systems that rely on the MOQ protocol.

## Specific Protocol Considerations {#proto-cons}

In order to support the various topologies and patterns of media flows with the protocol, the protocol MUST support both sending and receiving of media streams, as separate actions or concurrently in a given connection.

### QUIC Capabilities and Properties

With QUIC being the underlying protocol brings capabilities and functionalities for many of the requirements such as connection migration and re-use, greater controls over packet reliability, congestion control, re-ordering and flow directionality, multiplexing and head of line blocking. Utilizing aspects of the QUIC protocol which would then necessitate re-implementation of these capabilities already present in other parts of the QUIC protocol should only be done so if requirements deem them incompatible.

### Delivery Assurance vs. Delay

Different use cases have varying requirements with respect to the tradeoffs associated in having guarantee of delivery vs delay - in some (such as telephony) it may be acceptable to drop some or all of the media as a result of changes in network connectivity, throughput, or congestion whereas in other scenarios all media must arrive at the receiving end even if delayed. There SHOULD be support for some means for a connection to signal which media may be abandoned, and behaviors of both senders receivers defined when delay or loss occurs. Where multiple variants of media are sent, this SHOULD be done so in a way that provides pipelining so each media stream may be processed in parallel.

### Support WebTransport/Raw QUIC as Media Transport

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

Considerations should be made to addressing of individual audio/video frames as opposed to groups, in addition to how the model incorporates signalling of prioritization, media dependency, and cacheability to all entities.

## Publishing Media {#pub-media}

Many of the use cases have bi-directional flows of media, with clients both sending and receiving media concurrently, thus the protocol should have a unified approach in connection negotiation and signalling to send and received media both at the start and ongoing in the lifetime of a session including describing when flow of media is unsupported (e.g. a live media server signalling it does not support receiving from a given client).

In the initiation of a session both client and server must perform negotiation in order to agree upon a variety of details before media can move in any direction:

- Is the client authenticated and subsequently authorized to initiate a connection?
- What media is available, and for each what are the parameters such as codec, bitrate, and resolution etc?
- Can media move bi-directionally, or is it unidirectional only?

## Naming and Addressing Media Resources {#naming}

As multiple streams of media may be available for concurrent sending such as multiple camera views or audio tracks, a means of both identifying the technical properties of each resource (codec, bitrate, etc) as well as a useful identification for playback should be part of the protocol. A base level of optional metadata e.g. the known language of an audio track or name of participant's camera should be supported, but further extended metadata of the contents of the media or its ontology should not be supported.

### Scoped to an Origin/Domain, Application specific

### Allows Consumers Subscribing or Requesting for Data Matching by Name

## Packaging Media {#Packaging}

Packaging of media describes how raw media will be encapsulated. There are at a high level two approaches to this:

- Within the protocol itself, where the protocol defines the ancillary data required to decode each media type the protocol supports.
- A common encapsulation format such as there are advantages to using an existing generic media packaging format (such as CMAF {{CMAF}} or other ISOBMFF {{ISOBMFF}} subsets) which define a generic method for all media and handles ancillary decode information.

The working group must agree on which approach should be taken to the packaging of media, taking into consideration the various technical trade offs that each approach provides.

- If the working group decides to describe media encapsulation as part of the MOQ protocol, this will require a new version of the MOQ protocol in order to signal the receiver that a new media encapsulation format may be present.

- If the working group decides to use a common encapsulation format, the mechanisms within the protocol SHOULD allow for new encapsulation formats to be used. Without encapsulation agility, adding or changing the way media is encapsulated will also require a new version of the MOQ protocol, to signal the receiver that a new media encapsulation format may be present.

MOQ protocol specifications will provide details on the supported media encapsulation(s).

### Handling Scalable Video Codecs

Some video codecs have a complex structure. Consider an
application using both temporal layering and spatial layering. It would
send for example:

- an object representing the 30 fps frame at 720p
- an object representing the spatial enhancement of that frame to 1080p
- an object representing the 60 fps frame at 720p
- an object representing the spatial enhancement of that 60 fps frame to 1080p

The encoding of the 30 fps frame depends on the previous 30 fps frames, but not on any 60 fps frame. The encoding of the 60 fps depends on the previous 30 fps frames, and possibly also on the previous 60 fps frames (there are options). The encoding of the spatial enhancement depends on the corresponding 720p frames, and also on the previous 1080p enhancements. Add a couple of layers, and the expression of dependencies can be very complex. The AV1 documentation for example provides schematics of a video stream with 3 frame rate options at 15, 30 and 60 fps, and two definition options, with a complex graph of dependencies. Other video encodings have similar provisions. They may differ in details, but there are constants: if some object is dropped, then all objects that have a dependency on it are useless.

Of course, we could encode these dependencies as properties of the object being sent, stating for example that "object 17 can only be decoded if objects 16, 11 and 7 are available." However, this approach leads to a lot of complexity in relays. We believe that a linear approach is preferable, using attributes of objects like delivery order or priorities.

### Application Choice for Ordering

The conversion from dependency graph to linear ordering is not unique. The simple graph in our example could be ordered either "frame rate first" versus "definition first". If the application chooses frame rate first, the policy is expressed as "in case of congestion, drop the spatial enhancement objects first, and if that is not enough drop the 60 fps frames". If the application chooses "definition first", the policy becomes "drop the 60 fps frames and their corresponding 1080p enhancement first, and if that is not enough also drop the 1080p enhancement of the 30 fps frames".

More complex graphs will allow for more complex policies, maybe for example "15 fps at 720p as a minimum, but try to ensure at least 30fps, then try to ensure 1080p, and if there is bandwidth available forward 60 fps at 1080p". Such linearization requires choices, and the choices should be made by the application, based on the user experience requirements of the application.

The relays will not understand all the variation of what the media is but the applications will need a way to indicate to the relays the information they will need to correctly order which data is sent first.

### Linear Ordering using Priorities

We propose to express dependencies using a combination of object number and object priority.

Let's consider our example of an encoding providing both spatial enhancement and frame rate enhancement options, and suppose that the application has expressed a preference for frame rate. We can express that policy as follow:
- the frames are ordered first by time and when the time is the same by resolution. This determines the "object number" property.
- the frame priority will be set to 1 for the 720p 30 fps frame, 2 for the 720p 60 fps frames, and 3 for all the enhancement frames.

If the application did instead express a preference for definition, object numbers will be assigned in the same way, but the priorities will be different:

- the frame priority will be set to 1 for the 720p 30 fps I frames and 2 for the 720p 30 fps P and B frames, 3 and 4 for the 1080p enhancements of the 60 fps frames, and 5 and 6 for the 60 fps frames and their enhancements.

Object numbers and priorities will be set by the publisher of the track, and will not be modified by the relays.

## Media Consumption {#med-consumption}

Receivers SHOULD be able to as part of negotiation of a session {{MOQ-negotiation}} specify which media to receive, not just with respect to the media format and codec, but also the variant thereof such as resolution or bitrate.

## Relays, Caches, and other MOQ Network Elements {#MOQ-network-entities}

### Intervals and Congestion

It is possible to use groups as units of congestion control. When the sending strategy is understood, the objects in the group can be assigned sequence numbers and drop priorities that capture the encoding dependencies, such that:

- an object can only have dependencies with other objects in the same group,
- an object can only have dependencies with other objects with lower sequence numbers,
- an object can only have dependencies with other objects with lower or equal drop priorities.

This simple rules enable real-time congestion control decisions at relays and other nodes. The main drawback is that if a packet with a given drop priority is actually dropped, all objects with higher sequence numbers and higher or equal drop priorities in the same group must be dropped. If the group duration is long, this means that the quality of experience may be lowered for a long time after a brief congestion. If the group duration is short, this can produce a jarring effect in which the quality of experience drops periodically at the tail of the group.

### Pull & Push

To enable use cases where receivers may wish to address a particular time of media in addition to having the most recently produced media available, both "pull" and "push" of media SHOULD be supported, with consideration that producers and intermediates SHOULD also signal what media is available (commonly referred to as a "DVR window"). Behaviors around cache durations for each MoQ entity should be defined.

### Relay Behavior

In case of congestion, the relay will use the priorities to selectively drop the "least important" objects:

- if congestion is noticed, the relay will drop first the lesser priority layer. In our example, that would mean the objects marked at priority 6. The relay will drop all objects marked at that priority, from the first dropped object to the end of the group.

- if congestion persists despite dropping a first layer, the relay will start dropping the next layer, in our example the objects marked at priority 5.

- if congestion still persist after dropping all but the highest priority layer, the relay will have to close the group, and start relaying the next group.

When dropping objects within the same priority:

- higher object numbers in the same group, which are later in the group, are "less important" and more likely to be dropped than objects in the same group with a lower object number. Objects in a previous group are "less important" than objects in the current group and MAY be dropped ahead of objects in the current group.

The specification above assumes that the relay can detect the onset of congestion, and has a way to drop objects. There are several ways to achieve that result, such as sending all objects of a group in a single QUIC stream and making explicit action at the time of relaying, or mapping separate priority layers into different QUIC streams and marking these streams with different priorities. The exact solution will have to be defined in a draft that specifies transport priorities.

### High Loss Networks

Web conferencing systems are used on networks with well over 20% packet loss and when this happens, it is often on connections with a relatively large round trip times. In these situation, forward error correction or redundant transmissions are used to provide a reasonable user experience. Often video is turned off in. There are multiple machine learning based audio codecs in development that targeting a 2 to 3 Kbps rate.

This can result in scenarios where very small audio objects are sent at a rate of several hundreds packets per second with a high network loss rate.

### Interval between Access Points

In the streaming scenarios, there is an important emphasis on resynchronization, characterized by a short distance between "access points". This can be used for features like fast-forward or rewinding, which are common in non-real-time streaming. For real-time streaming experiences such as watching a sport event, frequent access points allow "channel surfers" to quickly join the broadcast and enjoy the experience. The interval between these access points will often be just a few seconds.

In video encoding, each access point is mapped to a fully encoded frame that can be used as reference for the "group of blocks". The encoding of these reference frames is typically much larger than the differential encoding of the following frames. This creates a peak of traffic at the beginning of the group. This peak is much easier to absorb in streaming applications that tolerate higher latencies than interactive video conferences. In practice, many real time conferences tend to use much longer groups, resulting in higher compression ratios and smoother bandwidth consumption along with a way to request the start of a new group when needed. Other real time conferences tend to use very short groups and just wait for the next group when needed.

Of course, having longer blocks create other issues. Realtime conferences also need to accommodate the occasional occasional late comer, or the disconnected user who want to resynchronize after a network event. This drives a need for synchronization "between access points". For example, rather than waiting for 30 seconds before connecting, the user might quickly download the "key" frames of the past 30 seconds and replay them in order to "synchronize" the video decoder.

### Media Insertion and Redirection

In all of the applicable use cases defined in {{overallusecases}} it may be necessary for consumers to be aware of changes to the source of media being inserted, or be instructed to consume media from a different source. These may be done for the insertion of advertising or for operational movement of consumers, amongst other reasons. Within the media insertion scenario an existing stream being consumed may change as a result of a different source being spliced which necessitates the decoder being reset as parameters such as video frame rate, image resolution etc may have changed. For redirection, consumers may be signalled to consume media from a different source which may also require re-initialization of decoder.

In both of these scenarios, triggering may occur either through an event provided in the media such as a {{SCTE-35}} marker, or through an external trigger. Both should be supported.

## Security {#MOQ-security}

### Authentication & Authorization

Whilst QUIC and conversely TLS supports the ability for mutual authentication through client and server presenting certificates and performing validation, this is infeasible in many use cases where provisioning of client TLS certificates is unsupported or impractical. Thus, support for a primitive method of authentication between MoQ entities SHOULD be included to authenticate entities between one another, noting that implementations and deployments should determine which authorization model if any is applicable.

### Media Encryption {#MOQ-media-encryption}

Much of the early discussion about MOQ security was not entirely coherent. Some contributors pushed for "end-to-end security", and some contributors pushed for the ability of intermediate nodes to have sufficient visibility into media payloads to accomplish the responsibilities those intermediate nodes were given. Some contributors may have pushed for both, at various times. It is worthwhile to clarify what "security" means in a MOQ context.

Generally, there are three aspects of media security:

- Digital Rights Management, which refers to the authorization of receivers to decode a media stream.
- Sender-to-Receiver Media Security, which refers to the ability of media senders and receivers to transfer media while protected from unauthorized intermediates and observers, and
- Node-to-node Media Security, which refers to security when authorized intermediaries are needed to transform media into a form acceptable to authorized receivers. For example, this might refer to a video transcoder between the media sender and receiver.

"End-to-end security" describes the use of encryption of one or more media stream(s) over an end-to-end path, to provide confidentiality in the presence of any intermediates or observers and prevent or restrict ability to decrypt that media.

"Node-to-node security" refers to the use of encryption of one or more media stream(s) over a path segment connecting two MOQ nodes, that makes up part of the end-to-end path between the MOQ sender and ultimate MOQ receiver, to provide confidentiality in the presence of unauthorized intermediates or observers and prevent or restrict ability to decrypt that media.

Many MOQ deployment models rely on intermediate nodes, and these intermediate nodes may have a variety of responsibilities, including, but not limited to,

- rate adaptation based on media metadata
- routing media based on the characteristics of the media
- caching media
- allowing "watch in-progress broadcasts from the beginning", "instant replay" and "fast forward"
- transcoding media

Some of these responsibilities require authorization to see more media headers and even media payload than others. The protocol SHOULD allow MOQ intermediate nodes to perform a variety of responsibilities, without having access to media headers and/or media payloads that they do not require to carry out their responsibilities.

Support for encrypted media SHOULD be available in the protocol to support the above use cases, with key exchange and decryption authorization handled externally. The protocol SHOULD provide metadata for entities which process media to perform key exchange and decrypt.

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to guide discussion and consensus, it introduces no security considerations of its own.

--- back

# Acknowledgements

A significant amount of material in {{overallusecases}} and {{req-sec}}} was taken from {{?I-D.draft-nandakumar-moq-scenarios}}. We thank the authors of that draft, Suhas Nandakumar, Christian Huitema, and Cullen Jennings.

The authors would like to thank several authors of individual drafts that fed into the "Media Over QUIC" charter process:

- Kirill Pugin, Alan Frindell, Jordi Cenzano, and Jake Weissman ({{I-D.draft-kpugin-rush}},
- Luke Curley ({{I-D.draft-lcurley-warp}}), and
- Cullen Jennings and Suhas Nandakumar ({{I-D.draft-jennings-moq-quicr-arch}}), together with Christian Huitema ({{I-D.draft-jennings-moq-quicr-proto}}).

We would also like to thank Suhas Nandakumar for his presentation, "Progressing MOQ" {{Prog-MOQ}}, at the October 2022 MOQ virtual interim meeting. We used his outline as a starting point for the Requirements section ({{req-sec}}).

We would also like to thank Cullen Jennings for suggesting that we distinguish between interactive and live streaming use cases based on the users' perception, rather than quantitative measurements. In addition we would also like to thank Lucas Pardue, Alan Frindell, and Bernard Aboba for their reviews of the document.

James Gruessing would also like to thank Francesco Illy and Nicholas Book for their part in providing the needed motivation.
