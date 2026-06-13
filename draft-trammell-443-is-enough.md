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
area: "Web and Internet Transport (WIT)"
# workgroup:
keyword:
 - HTTP
 - port allocation
 - IANA
 - service registration

venue:
  github: "britram/draft-trammell-443-is-enough"
  latest: "https://britram.github.io/draft-trammell-443-is-enough/draft-trammell-443-is-enough.html"

author:
 -
    fullname: Brian Trammell
    email: ietf@trammell.ch

normative:
  RFC7605:
  RFC9205:

informative:
  RFC7301:
  RFC8615:
  RFC9000:
  RFC9110:
  RFC9113:
  RFC9114:
  RFC6455:

...

--- abstract

{{RFC7605}} provides guidance on the use of port numbers and the criteria for
new port assignments, including a test for whether a proposed service is
distinct from an existing service. It gives the example that "an automated
system that happens to use HTTP framing -- but is not primarily accessed by a
browser -- might be a new service." It also might not. This document clarifies
the application of the distinct-protocol test in {{RFC7605}} Section 7.1 to
services built on HTTP as a substrate, in light of HTTP's evolution since its
publication, and provides guidance to applicants and reviewers on when an
HTTP-based service qualifies for a new port assignment and when it does not.

--- middle

# Introduction

{{RFC7605}} provides guidance on when a new port assignment is warranted,
including a distinctness test in Section 7.1: a new service merits an
assignment only if an unmodified client of an existing service cannot
interact with it.

In the decade since that document was published in 2015, HTTP has become an
overwhelming popular de facto substrate for application protocol design -- a
development that {{RFC9205}} both documents and embraces. Section 7.1's
observation that "an automated system that happens to use HTTP framing -- but is
not primarily accessed by a browser -- might be a new service" was intended to
leave room for novel cases. The evolution of and investment in the HTTP
ecosystem since then has only made the use of HTTP and the ecosystem surrounding
it as a substrate more attractive. One practical consequence of this development
has been some confusion about whether new protocols over HTTP are new protocols
in the sense of "requiring a port assignment".

This document clarifies how the {{RFC7605}} Section 7.1 distinctness test
applies to HTTP-based services, and provides guidance to applicants and
reviewers on when it is and is not satisfied.

# HTTP as an Application Transport Substrate

HTTP has evolved since its origins as the basis of the World Wide Web. HTTP/2
{{RFC9113}} redesigned HTTP's wire format around multiplexed binary framing with
non-browser use as an explicit design goal; HTTP/3 {{RFC9114}} continues this
evolution over QUIC {{RFC9000}}. {{RFC9205}} provides detailed guidance on
building new protocols beyond the web atop HTTP. The benefits of this approach
are substantial: HTTP-based services can leverage existing infrastructure
including reverse proxies, load balancers, content delivery networks, and
firewalls; they interoperate naturally with web clients; and they inherit
well-established security properties including TLS certificate management and
authentication frameworks.

The HTTP ecosystem also provides a rich set of mechanisms for service
differentiation and discovery that do not require dedicated port assignments,
such as Application-Layer Protocol Negotiation (ALPN) {{RFC7301}} and Well-Known
URIs {{RFC8615}} supporting service discovery at different points in the
protocol handshake. A service that requires a new ALPN identifier should
register it in the IANA TLS ALPN Protocol IDs registry, not seek a new port
assignment.

# Evaluating HTTP-Based Protocols for Distinctness

Section 7.1 of {{RFC7605}} establishes one useful test for whether a proposed
service warrants a new port assignment: can an unmodified client of an existing
service interact with the proposed service? Interoperability implies
non-distinctness, and a non-distinct protocol does not merit a new assignment.

For HTTP-based services, this test is easy to implement: can an unmodified
generic HTTP client tool such as curl issue requests to and receive valid
responses from the proposed service? Service differentiation achieved through
URL path structure, HTTP header values, Content-Type negotiation, payload
schema, or authentication scheme does not constitute wire-level distinctness;
these are application-layer conventions carried within HTTP, not independent
protocols.

This does not mean that all HTTP-based protocols are indistinct. Examples that
might warrant an assignment include: a REST API running over the same TLS
connection as a protocol with a different wire format, using a protocol-specific
multiplexing scheme; or a protocol running on UDP or SCTP that uses a REST API
over TCP as a control or management plane. The former would not interoperate
with an unmodified client, and the latter has incomplete semantics when used as
such.

# Security Considerations

The intended effect of the guidance given by this document is effectively to
reduce port assignments for HTTP-based services, directing these services to use
port 443 rather than dedicated port assignments. This has implications for
overall network security: traffic from non-Web HTTP applications running on port
443 is less distinguishable from other traffic in the face of metadata
examination. TLS deployment is more likely to be properly configured when
services share the standard HTTPS port and its associated certificate management
infrastructure, and network operators can apply consistent security policy
across all services on that port. {{RFC9205}} Section 4.4.3 notes that deploying
an HTTP-based application on a non-default port carries privacy implications
because the protocol becomes distinguishable from other traffic; the guidance in
this document is consistent with minimizing that distinguishability.

# IANA Considerations

This document has no IANA actions. It is intended as guidance for IANA Transport
Port Expert Reviewers.

--- back

# Disclosure
{:numbered="false"}

LLM-based tools (Claude Code, Sonnet 4.6) were used in the workflow management,
reference and archival research, initial draft generation, and editorial review
of this document.
