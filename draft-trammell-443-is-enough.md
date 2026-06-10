---
title: "443 is Enough: Guidance on Port Allocation for HTTP-based Services"
abbrev: "443 is Enough"
category: info

docname: draft-trammell-443-is-enough-latest
submissiontype: independent
number:
date:
consensus: false
v: 3
area: "Applications and Real-Time"
workgroup:
keyword:
 - HTTP
 - port allocation
 - IANA
 - service registration

venue:
  github: britram/draft-trammell-443-is-enough
  latest: https://britram.github.io/draft-trammell-443-is-enough/draft-trammell-443-is-enough.html

author:
 -
    fullname: Brian Trammell
    email: ietf@trammell.ch

normative:
  RFC7605:
  RFC9110:
  RFC9113:
  RFC7301:
  RFC9205:

informative:
  RFC8615:
  RFC9114:
  RFC6455:

...

--- abstract

{{RFC7605}} provides guidance on the use of port numbers and the criteria for
new port assignments, including a test for whether a proposed service is
distinct from an existing service. It gives the example that "an automated
system that happens to use HTTP framing -- but is not primarily accessed by a
browser -- might be a new service." It also might not. This document clarifies
the application of the RFC 7605 distinctness test to HTTP-based services, in
light of HTTP's evolution since 2015 into a general-purpose application
transport substrate, and provides guidance to applicants and reviewers on when
an HTTP-based service qualifies for a new port assignment and when it does not.

--- middle

# Introduction

TODO

# HTTP as an Application Transport Substrate

TODO

# Applying the RFC 7605 Distinctness Test to HTTP-based Services



# When HTTP-based Services Do Not Qualify for Port Assignment

TODO

# When HTTP-based Services May Qualify for Port Assignment

TODO

# Alternative Registration Mechanisms

TODO

# Security Considerations

TODO

# IANA Considerations

This document has no IANA actions.

--- back

# Notes
{:numbered="false"}

The following notes record the analysis behind this document and are
intended to guide drafting. They are not part of the final document.

## Context

RFC 7605 section 7.1 says an "automated system that happens to use HTTP
framing -- but is not primarily accessed by a browser -- might be a new
service." That sentence was written in 2015, when it was still reasonable
to treat HTTP as primarily a browser protocol with occasional non-browser
uses. It no longer reflects the landscape. HTTP/2 (RFC 7540, now RFC
9113) was explicitly designed as a general-purpose application transport
substrate -- multiplexed streams, binary framing, header compression --
with non-browser use as a first-class design goal. HTTP/3 (RFC 9114)
continues that trajectory over QUIC. The dominant application protocol
design pattern of the 2010s-2020s is: run your protocol over HTTP, use
TLS on port 443, differentiate by URL path and Content-Type. gRPC,
GraphQL, OpenAPI-described REST APIs, OCI registries, CalDAV, CardDAV --
all are HTTP profiles. The section 7.1 "might be a new service" sentence
was intended to leave room for this pattern where the wire-level protocol
is genuinely distinct; in practice it has become the opening every HTTP
API applicant reaches for.

## Relationship to RFC 9205

RFC 9205 ("Building Protocols with HTTP", Nottingham, 2022) is the IETF's
canonical guidance on using HTTP as a substrate for application protocols.
It defines what it means for a protocol to "use HTTP" (including an
ALPN-based criterion), and addresses port selection -- but more permissively
than our argument requires. RFC 9205 §4.4.3 says applications "can use the
applicable default port (80 for HTTP, 443 for HTTPS), or they can be
deployed upon other ports," and notes that port registration is a legitimate
way to encourage a specific choice.

This draft complements RFC 9205 rather than contradicting it. RFC 9205
answers "how do you build a good HTTP-based protocol?"; this document
answers "does building a good HTTP-based protocol entitle you to a port
number?" The answer to the second question is no -- and the reason is RFC
7605, not RFC 9205. The draft should cite RFC 9205 in the introduction,
acknowledge that it permits port registration for HTTP-based applications,
and explain that the RFC 7605 distinctness test is the binding constraint
that RFC 9205 does not address.

RFC 6455 (WebSocket) should be cited as an informative example of the right
pattern: a protocol that bootstraps over HTTP, becomes wire-distinct after
the Upgrade handshake (no unmodified HTTP client can exchange WebSocket
frames), and uses an ALPN identifier -- but not its own port. It runs on 443
as wss:// alongside HTTPS.

## The Clarifying Test

The document needs to state clearly what "might be a new service" actually
requires. The relevant question is not "is this accessed by a browser?"
but "is the wire protocol distinct from HTTP?" A service is an HTTP
profile -- and does not qualify for a new port assignment -- if all of the
following hold:

- An unmodified HTTP client (operationalised: curl with appropriate
  headers) can issue requests and receive structurally valid responses.

- Service differentiation from other HTTP services is achieved via URL
  path structure, HTTP headers, Content-Type values, JSON schema, or
  authentication scheme -- all of which are application-layer conventions
  within HTTP, not wire-level distinctions.

- No transport-layer behaviour is required that falls outside standard
  HTTP semantics: the service does not require a UDP component, a
  non-HTTP framing layer, or an ALPN identifier distinct from h2, h2c,
  or http/1.1.

## The Escape Valves

The document should also define when an HTTP-based service does qualify.
Three cases:

1. The content encoding is genuinely opaque to a generic HTTP client -- a
   binary MIME type with a format no standard client library can parse
   without the application-specific codec. gRPC's application/grpc with
   protobuf is the canonical example; note that gRPC itself uses port 443
   with ALPN, not its own port.

2. The service has a substantial non-HTTP transport component on the same
   port (e.g., DTLS/UDP telemetry co-registered with the TCP API), where
   the combined service cannot be fully accessed by any unmodified HTTP
   client. The non-HTTP component must be implemented and specified, not
   merely claimed.

   An important corollary: when a protocol is primarily non-HTTP (e.g., a
   UDP or SCTP protocol) and carries a REST API as a management or
   control bolt-on over the same port, the REST surface does not disqualify
   the registration. Denying the port in such a case would force the most
   natural client behavior -- connecting to the well-known port -- to be
   squatting, which is contrary to the intent of RFC 7605 §7.8. The
   registration is for the combined service; the REST component follows
   the port, not the other way around.

3. The service requires an ALPN identifier not already registered -- in
   which case the right registration is in the IANA TLS ALPN Protocol IDs
   registry (RFC 7301), not a new port.

## Alternative Mechanisms

For applicants whose service is an HTTP profile, the document should name
the better-suited mechanisms:

- ALPN registration (RFC 7301) for protocol-level differentiation under
  TLS.

- Well-Known URIs (RFC 8615) for service discovery.

- HTTP virtual hosting via Host header / TLS SNI for co-location of
  multiple services on the same port.

These mechanisms were designed specifically to support the ecosystem of
HTTP-based services without consuming the 16-bit port number space.

## Form and Venue

This should be a short Informational RFC (target 4-6 pages formatted),
submitted via the independent stream. It does not need working group
adoption to be useful -- it just needs to be stable and citable. An
Internet-Draft posted to the IETF datatracker achieves the "citable
stable document" goal even before publication, which is sufficient for
the port reviewer use case.

The framing should read as clarification of what RFC 7605 section 7.1
always meant, not as new policy. The unmodified-client test has always
been the rule; this document explains how to apply it to the
HTTP-as-substrate world that section 7.1 did not anticipate needing to
address explicitly.

# Acknowledgments
{:numbered="false"}

TODO

