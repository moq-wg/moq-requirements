---
title: Media over QUIC Requirements and Use Cases
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

normative:
  RFC9000:
  I-D.draft-ietf-webtrans-overview:

informative:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-kpugin-rush:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-sharabayko-srt:
  RFC3550:

--- abstract

This document describes the uses cases, requirements, and considerations that
should be part of the design of a real-time media protocol over QUIC
{{RFC9000}}.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-moq-requirements>.

--- middle

# Introduction {#intro}

# Prior and Existing Specifications {#priorart}

Several existing draft specifications and protocols already exist which base
their implementation around using existing protocols on top of QUIC, or define
their own. With the exception of RUSH, it is unknown if the other specifications
have had any deployments or interop with multiple implementations.

## draft-hurst-quic-rtp-tunnelling

QRT encapsulates RTP and RTCP and define the means of using QUIC datagrams
with them, defining a new payload within a datagram frame which distinguishes
packets for a RTP packet flow vs RTCP.

## draft-engelbart-rtp-over-quic

This specification also encapsulates RTP and RTCP but unlike QRT which simply
relies on the default QUIC congestion control mechanisms, it defines a set of
requirements around QUIC implementation's congestion controller to permit the
use of separate control algorithms.

## draft-kpugin-rush

RUSH uses its own frame types on top of QUIC as it pre-dates the datagram
specification; in addition individual media frames are given their own stream
identifiers to remove HoL blocking from processing out-of-order.

It defines its own registry for signalling codec information with room for
future expansion but presently is limited to a subset of popular video and audio
codecs and doesn't include other types (such as subtitles, transcriptions, or
other signalling information) out of bitstream.

## draft-sharabayko-srt-over-quic

SRT {{I-D.draft-sharabayko-srt}} itself is a general purpose transport protocol
primarily for contribution transport use cases and this specification covers the
encapsulation and delivery of SRT on top of QUIC using datagram frame types.
This specification sets some requirements regarding how the two interact and
leaves considerations for congestion control and pacing to prevent conflict
between the two protocols.

## Comparison of Existing Specifications

* Both QRT and the Engelbart draft attempt to use existing payloads of RTP,
  RTCP, and SDP unlike RUSH and SRT, as well as using existing Datagram frames
* RUSH introduces new frame types as its development pre-dates Datagram frames
* Both QRT and RUSH specify ALPN identification; the Engelbart and SRT drafts do not.
* All drafts take differing approaches to flow/stream identification and
  management; some address congestion control and others just omit the subject
  and leave it to QUIC to handle

# Use Cases {#usecases}

Previously {{I-D.draft-rtpfolks-quic-rtp-over-quic}} defined several key use
cases, in addition to several others defined elsewhere. The two use cases that
are most applicable today given the existing and known future capabilities of
QUIC include:

1. Unidirectional live stream contribution. Two immediate scenarios that
best describe this is firstly users on a streaming platform in a remote scenario
from their phone live streaming an event or going on to an audience in real time
in relatively low bitrates (~1-5Mbit). The second scenario is larger bitrate
contribution feeds in broadcast. This can be an OB feed "back to base" into
playout gallery, or from playout facilities to online distribution platforms.

2. Distribution from platform to audience. Whilst use of WebRTC or RTSP today
for On-Demand media streaming is not typical with adaptive streaming like HLS
and DASH being predominantly used as WebRTC is more applicable in latency
sensitive contexts such as live sporting events. Instead use cases where there
is live streaming of TV linear output, or live streaming such as Twitch or
Facebook, or non-UGC services like OTT offerings made by broadcasters.

# Requirements {#requirements}

This section lists requirements for providing real time media streaming over a
QUIC connection.

## Codec Agility

When initiating a media session, both the sender and receiver should be able to
negotiate the codecs, bitrates and other media details based on capabilities and
preferences.

## Flow Directionality

Media should be able to flow in either direction from client to server or
vice-versa, either individually or concurrently.

## WebTransport

TODO: Unsure if this should be a requirement. If it is, we have to consider two
things: WebTransport supports HTTP/2, are we going to explicitly exclude it?
Also, WebTransport {{I-D.draft-ietf-webtrans-overview}} has normative language
around congestion control which may be at odds with our potential requirements.

## Authentication

The protocol SHOULD have capabilities asides from TLS mutual authentication to
allow hosts to authenticate one another, this should be kept simple but robust
in nature to prevent attacks like credential brute-forcing.

# Non-requirements

This section covers topics that are explicitly out of scope for the time being.

## Media Transport Protocols

The creation of new media transport protocols should be avoided, and instead we
should make use of RTP {{RFC3550}} and the existing ecosystem of payload formats
and methods of signalling where possible. It may transpire a need to extend
these specificiations; in which case we should work with the relevant working
groups and present our use-cases.

## Multicast

Even if multicast and other network broadcasting capabilities are used in
delivering media in our use cases, as QUIC doesn't yet support it and it's
inclusion would require a lot more complexity in both the specification and
client implimentation this should be left out for now.

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to create discussion and consensus and introduces
no security considerations of its own.

--- back

# Acknowledgements

The author would like the thank the many draft authors of various specifications
for their work. The author would also like to thank Francesco Illy and Nicholas
Book for their part in providing the needed motivation.
