---
title: Media over QUIC Requirements and Use Cases
abbrev: MoQ Requirements and Use Cases
docname: draft-gruessing-moq-requirements-latest
category: info

author:
  - ins: J. Gruessing
    name: James Gruessing
    email: james.ietf@gmail.com

ipr: trust200902
area: applications
workgroup: AVTCORE/MMUSIC Working Groups
keyword: Internet-Draft QUIC RTP
stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, subcompact, comments, inline]

normative:
  RFC9000:

informative:
  I-D.draft-rtpfolks-quic-rtp-over-quic:
  I-D.draft-hurst-quic-rtp-tunnelling:
  I-D.draft-engelbart-rtp-over-quic:
  I-D.draft-kpugin-rush:
  I-D.draft-sharabayko-srt-over-quic:
  I-D.draft-sharabayko-srt:

--- abstract

This document describes the uses cases, requirements, and considerations that
should be part of the design of a real-time media protocol over QUIC
{{RFC9000}}.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/I-D/tree/main/draft-gruessing-moq-requirements>.

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

For the sake of completeness this is a description of the known use-cases that
have been described which would have applicability to the real-time serving of
media over QUIC, and MoQ participants should consider which use case(s) should
be part of any work, and which can be excluded. Previously
{{I-D.draft-rtpfolks-quic-rtp-over-quic}} defined several key use cases, in
addition to several others which may be summarised under the following:

1. Peer-to-peer interactive applications, such as telephony or video
conferencing. This may be in a 1-to-1 scenario, or in a multi-party arrangement.
TODO: add more description here.

2. Unidirectional live stream contribution. Two immediate scenarios that
best describe this is firstly users on a streaming platform in a remote scenario
from their phone live streaming an event or going on to an audience in real time
in relatively low bitrates (~1-5Mbit). The second scenario is larger bitrate
contribution feeds in broadcast. This can be an OB feed "back to base" into
playout gallery, or from playout facilities to online distribution platforms.

3. Distribution from platform to audience. Whilst use of WebRTC or RTSP today
for On-Demand media streaming is not typical with adaptive streaming like HLS
and DASH being predominantly used as WebRTC is more applicable in latency
sensitive contexts such as live sporting events. Instead use cases where there
is live streaming of TV linear output, or live streaming such as Twitch or
Facebook, or non-UGC services like OTT offerings made by broadcasters.

# Requirements {#requirements}

TODO: Fill this in with detail

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
