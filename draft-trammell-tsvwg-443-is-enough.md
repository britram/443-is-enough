---
title: "443 is Enough: Guidance on Port Allocation for HTTP-based Services"
abbrev: "443 is Enough"
category: info

docname: draft-trammell-tsvwg-443-is-enough-latest
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Transport and Services Working Group"
keyword:
 - HTTP
 - port allocation
 - IANA
 - service registration

venue:
  github: "britram/443-is-enough"
  latest: "https://britram.github.io/443-is-enough/draft-trammell-tsvwg-443-is-enough.html"

author:
 -
    fullname: Brian Trammell
    org: Google Switzerland GmbH
    email: ietf@trammell.ch
    street: Gustav-Gull-Platz 1
    code: 8004
    city: Zürich
    country: Switzerland

normative:
  RFC7605:
  RFC9205:

informative:
  RFC2782:
  RFC6066:
  RFC7301:
  RFC8484:
  RFC8552:
  RFC8615:
  RFC9000:
  RFC9110:
  RFC9113:
  RFC9114:
  RFC9250:
  RFC6455:
  I-D.ietf-webtrans-http3:

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
including a distinctness test in Section 7.1: a new service merits an assignment
only if an unmodified client of an existing service cannot interact with it.

In the decade since that document was published in 2015, HTTP has become an
overwhelmingly popular de facto substrate for application protocol design -- a
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
reviewers on when it is and is not satisfied. It does not replace {{RFC7605}},
but rather updates its application to reflect the maturity of the HTTP
ecosystem. Specifically, it addresses how modern deployment patterns—such as
ubiquitous TLS, SNI, ALPN, and user-space demultiplexing—have resolved the
practical cohabitation and access control issues that have previously motivated
requests for dedicated port assignments.

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

Operating on standard web ports (80 and 443) also improves compatibility with
existing network tooling—such as packet analyzers and diagnostic tools that are
pre-configured for HTTP—and maximizes the likelihood of traversing firewalls
that restrict outbound traffic to standard web ports, as is common in enterprise
and public access networks. This matters most for applications that will be
used in arbitrary network environments. Such traversal is not
guaranteed: firewalls increasingly apply deep packet inspection and
application-behavior analysis to traffic on ports 80 and 443, so using these
ports offers the best chance of traversal rather than a certainty of it.
Furthermore, reusing these ports directly supports transport port conservation,
a key goal of {{RFC7605}}.

The HTTP ecosystem also provides a rich set of mechanisms for service
differentiation, discovery, and coexistence that do not require dedicated port
assignments. Multiple independent services can share ports 80 and 443
concurrently on the same host using path-based routing (via reverse proxies or
API gateways) or host-based routing (via TLS Server Name Indication (SNI)
{{RFC6066}}), while Application-Layer Protocol Negotiation (ALPN) {{RFC7301}}
selects among versions of HTTP on the same port. These user-space demultiplexing
techniques are standard in modern deployments, resolving the "first binder wins"
problem inherent in OS-level transport-layer demultiplexing. Additionally,
sharing these ports allows network operators to leverage Layer 7 security
policies (such as SNI-based filtering) rather than relying on port-based
firewall rules.

An HTTP-based service uses the ALPN protocol identifiers that generically
identify HTTP ("http/1.1", "h2", "h3"), and needs neither a new port nor a new
ALPN identifier. A protocol that needs an ALPN identifier of its own is, by that
fact, not HTTP; see {{distinctness}}.

# Evaluating HTTP-Based Protocols for Distinctness {#distinctness}

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

This test tracks the scope of {{RFC9205}}: an application that uses HTTP's
ports, URI schemes, or ALPN identifiers is using HTTP, and must meet all of
HTTP's requirements. An application that passes the test does. One that relies
on HTTP but changes its operation, for example with application-specific
methods, does not; {{RFC9205}} directs it to establish its own port, and it may
merit an assignment. Examples that might warrant an assignment include:

-   a REST API running over the same TLS connection as a protocol with a
    different wire format, using a protocol-specific multiplexing scheme; or

-   a protocol running on UDP or SCTP that uses a REST API over TCP as a control
    or management plane.

The former would not interoperate with an unmodified client, and the latter has
incomplete semantics when used as such.

Further cases illustrate the boundary between "warrants an assignment" and not:

-   A protocol running directly over QUIC {{RFC9000}}, rather than through the
    HTTP/3 mapping {{RFC9114}}, is distinct from HTTP even though it shares a
    transport with it. DNS-over-QUIC (DoQ) {{RFC9250}} runs directly over QUIC
    and is assigned port 853; DNS-over-HTTPS (DoH) {{RFC8484}} carries DNS
    within HTTP on standard web ports.

-   ALPN makes it technically possible for such a protocol to share port 443
    with HTTP. However, {{RFC9205}} treats an application on port 443 as using
    HTTP, which such a protocol is not. That a protocol could be demultiplexed
    by ALPN on port 443 is not a reason to deny it a port for which it
    otherwise qualifies under {{RFC7605}}.

-   A protocol carried within a WebSocket {{RFC6455}} or WebTransport
    {{I-D.ietf-webtrans-http3}} session uses HTTP only to set up the session,
    then exchanges messages of its own. It nevertheless belongs on port 443:
    the session is established by an HTTP request to an "https" URI and routed
    like any other HTTP resource, so what appears on the port is HTTP, and the
    carried protocol is application content, like a payload schema. These
    mechanisms are designed to run on HTTP's ports (Section 1.7 of {{RFC6455}})
    so that clients, including Web browsers, can reach a server wherever HTTP
    can. The carried protocol is identified by a WebSocket subprotocol name
    (Section 11.5 of {{RFC6455}}) or through WebTransport's application protocol
    negotiation (Section 3.3 of {{I-D.ietf-webtrans-http3}}), not by a port.

-   A protocol defined both over HTTP and natively over QUIC, such as a media
    delivery protocol that can run either over WebTransport or directly over
    QUIC, is two cases: the variant over HTTP uses port 443, as above, and the
    native variant is evaluated on its own merits.

-   A protocol whose UDP component warrants its own port, but which uses an
    HTTP-based REST API for control, management, or bootstrap, should weigh
    using the corresponding TCP port for the control plane against the
    advantages of standard web ports.

# Application Naming Beyond Port Numbers

A service built on these substrates may be identified in more than one IANA
registry: a service name in the Service Name and Transport Protocol Port Number
Registry (used, for example, in DNS SRV records {{RFC2782}}); an ALPN protocol
identifier in the TLS Application-Layer Protocol Negotiation (ALPN) Protocol IDs
registry {{RFC7301}}; and an underscored node name in the Underscored and
Globally Scoped DNS Node Names registry {{RFC8552}}. These registries evolved
independently and are not coordinated. A single service may require entries in
several of them, with no guarantee that a chosen name is available or consistent
across all three. Rationalizing these namespaces is out of scope for this
document; the point here is that a port assignment is only one of several forms
of registration a service may need, and often not the one most relevant to how
the service is actually selected on the wire.

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

LLM-based tools (Claude Code, Antigravity) were used in the workflow management,
reference and archival research, initial draft generation, and editorial review
of this document, in part as an evaluation of the readiness of these tools for
such tasks.

# Acknowledgments
{:numbered="false"}

The author would like to thank Wesley Eddy, Michael Scharf, Joe Touch, Christian
Huitema, Magnus Westerlund, Jonathan Lennox, and Lorenzo Colitti for the
input and feedback that improved this document.
