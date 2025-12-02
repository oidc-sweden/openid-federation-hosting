%%%
title = "OpenID Federation Entity Configuration Hosting 1.0 - draft 00"
abbrev = "openid-federation-hosting"
ipr = "none"
workgroup = "OpenID Connect A/B"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-federation-hosting"
status = "standard"

[[author]]
initials="M."
surname="Lindström"
fullname="Martin Lindström"
organization="IDsec Solutions"
    [author.address]
    email = "martin@idsec.se"

[[author]]
initials="S."
surname="Santesson"
fullname="Stefan Santesson"
organization="IDsec Solutions"
    [author.address]
    email = "stefan@idsec.se"

%%%

.# Abstract

This specification defines an extension to OpenID Federation that enables Subordinate Statements to include a Claim indicating where a subject’s Entity Configuration is located. The extension introduces the `ec_location` Claim, which specifies an alternative location for retrieving the subject’s Entity Configuration. This functionality allows Entity Configuration data to be hosted at an Intermediate Entity, which can be essential for enabling legacy systems or non-federation capable entities to register and operate within a federation.

{mainmatter}

# Introduction

OpenID Federation 1.0 [@!OpenID.Federation] defines a configuration endpoint where Federation Entities should publish their Entity Configuration. The URL for this endpoint is the Entity Identifier concatenated with `/.well-known/openid-federation`.

One situation where a Federation Entity may not be able to make their Entity Configuration available at this URL is when their Entity Configuration is hosted by a Superior Entity and published under a URL based on a domain name that is owned by the Superior Entity. Reasons for this may be that an Entity registering to a federation is not fully OpenID Federation-compatible, or simply that it cannot expose its Entity Configuration at this URL.

Another situation is when a Federation Entity, for legacy reasons, has an Entity Identifier that is not a URL. This is a case that is valid according for several Entity Types that may be supported by OpenID Federation.

Not publishing the Entity Configuration at the configuration endpoint is a choice by the Federation Entity itself with the clear consequence that the Federation Entity will not be discoverable based on their Entity Identifier alone.

This specification defines the `ec_location` Claim to be used in Subordinate Statements, holding a location where the Entity Configuration of the subordinate can be obtained.

This specification also describes an alternate method of constructing Trust Chains, where chains are constructed in a top-down fashion.

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in BCP 14 [@!RFC2119]
[@!RFC8174] when, and only when, they appear in all capitals, as shown here.

## Terminology

This specification uses the terms "Claim" and "JSON Web Token (JWT)" as defined by JSON Web Token (JWT) [@!RFC7519], and "Entity", "Entity Identifier", "Trust Anchor", "Federation Entity", "Entity Statement", "Entity Configuration", "Subordinate Statement", "Intermediate Entity", "Leaf Entity", "Subordinate Entity", "Superior Entity", and "Trust Chain" defined in OpenID Federation 1.0 [@!OpenID.Federation].

# The ec_location Claim {#the_ec_location_claim}

This section defines the `ec_location` Claim that MAY be included in a Subordinate Statement.

Using this Claim, an alternative location for the subject’s Entity Configuration can be specified in a Subordinate Statement. Typically, this location differs from the configuration endpoint defined in Section 9 of [@!OpenID.Federation].

The value of the `ec_location` Claim is a single string that specifies the URL where the subject’s Entity Configuration is available. This URL MUST use either the https scheme or the data URL scheme [@!RFC2397], containing the complete Entity Configuration as a signed JWT.

A data URL MUST use the media type `application/entity-statement+jwt` and MUST NOT include a `;base64` declaration. The Entity Configuration data MUST be provided using compact serialization without additional Base64 encoding.

Note: Base64 encoding is omitted because the Entity Configuration JWT already uses a URL-safe format.

Section 9 of [@!OpenID.Federation] mandates that Intermediate Entities and Trust Anchors publish their Entity Configuration at the configuration endpoint. Therefore, it is RECOMMENDED to use the `ec_location` Claim in Subordinate Statements only for Leaf Entities.

When the `ec_location` Claim is present in a Subordinate Statement, and the subject Entity does not publish its Entity Configuration at the configuration endpoint defined in Section 9 of [@!OpenID.Federation], this Claim MUST be marked as critical, see Section 13.4 of [@!OpenID.Federation].

```json=
{
  "iss" : "https://superior-entity.example.com",
  "sub" : "example-legacy",
  ...  
  "ec_location" : "https://superior-entity.example.com/hosted/ec/example-legacy",
  ...
  "crit" : [ "ec_location", ... ]
}
```

**Example 1**: Excerpt of the Subordinate Statement issued by `https://superior-entity.example.com` for the subject `example-legacy`, where the Intermediate Entity itself hosts the subject's Entity Configuration. This is indicated using the `ec_location` Claim.

```json=
  "ec_location" : "data:application/entity-statement+jwt,eyJhb...Qssw5c"
```

**Example 2**: Using a data URL as the value for `ec_location`.

# Trust Chain Construction

Section 10.1 of [@!OpenID.Federation] describes the process of establishing a Trust Chain using a bottom-up approach, where the process starts with the Entity Configuration of the Entity whose metadata and Trust Chain are to be resolved, and then constructs a chain up to a trusted Trust Anchor. This process uses the `authority_hints` Claim of each fetched Entity Statement to identify the next Superior Entity. The procedure is repeated until at least one trusted Trust Anchor is reached for which a complete chain of statements can be evaluated.

Using the above process will not work unless a Leaf Entity publishes its Entity Configuration at the configuration endpoint as specified in Section 9 of [@!OpenID.Federation]. Not all entities within a federation may be able to adhere to this requirement, and therefore, alternative chain construction mechanisms need to be defined.

This specification (see (#top_down_chain_building) below) defines a top-down chain building mechanism that does not presuppose that all Leaf Entities publish their own Entity Configuration. This mechanism also makes use of the `ec_location` Claim as specified in (#the_ec_location_claim).

## Top-down Chain Building {#top_down_chain_building}

Top-down chain building starts with the Entity Configuration of a Trust Anchor and constructs a data map of all valid chain paths under this Trust Anchor. What is considered a valid chain is determined by the path builder based on any suitable criteria including, but not limited to, constraints, entity type, and similar factors.

The process used to build valid paths is to traverse all Intermediate Entities under the Trust Anchor and to list all Subordinate Entities using their subordinate listings endpoint (see Section 8.2 of [@!OpenID.Federation]). This process is repeated until all relevant Leaf Entities have been mapped.

The most important difference between top-down and bottom-up chain construction, as defined in Section 10.1 of [@!OpenID.Federation], is the order in which Entity Statements are collected. In top-down construction, the Subordinate Statement of an Entity is fetched before the Entity Configuration for the same subject, and this fact, together with the use of the `ec_location` Claim ((#the_ec_location_claim)), enables the construction of a chain even when a subject Entity does not publish its Entity Configuration at the configuration endpoint.

Federations that rely on the use of Resolvers (see Section 10.6 of [@!OpenID.Federation]) implementing top-down chain construction can use this capability to enable hosting of Entity Configuration documents at more convenient locations, while still ensuring the availability of validated peer entity data via resolve endpoints.


# Implementation Considerations

TODO

# Security Considerations

TODO

# Privacy Considerations

TODO


# Acknowledgments

We would like to thank the following individuals for their comments, ideas, and contributions to this implementation profile and to the initial set of implementations.

TODO

{backmatter}

<reference anchor="OpenID.Core" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 2</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="C." surname="Mortimore" fullname="Chuck Mortimore">
      <organization>Salesforce</organization>
    </author>
   <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="OpenID.Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html">
  <front>
    <title>OpenID Connect Discovery 1.0</title>

    <author fullname="Nat Sakimura" initials="N." surname="Sakimura">
      <organization abbrev="NAT.Consulting (was at NRI)">NAT.Consulting</organization>
    </author>

    <author fullname="John Bradley" initials="J." surname="Bradley">
      <organization abbrev="Yubico (was at Ping Identity)">Yubico</organization>
    </author>

    <author fullname="Michael B. Jones" initials="M.B." surname="Jones">
      <organization abbrev="Self-Issued Consulting (was at Microsoft)">Self-Issued Consulting</organization>
    </author>

    <author fullname="Edmund Jay" initials="E." surname="Jay">
      <organization abbrev="Illumila">Illumila</organization>
    </author>

    <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="OpenID.Registration" target="https://openid.net/specs/openid-connect-registration-1_0.html">
  <front>
    <title>OpenID Connect Dynamic Client Registration 1.0</title>

    <author fullname="Nat Sakimura" initials="N." surname="Sakimura">
      <organization abbrev="NAT.Consulting (was at NRI)">NAT.Consulting</organization>
    </author>

    <author fullname="John Bradley" initials="J." surname="Bradley">
      <organization abbrev="Yubico (was at Ping Identity)">Yubico</organization>
    </author>

    <author fullname="Michael B. Jones" initials="M.B." surname="Jones">
      <organization abbrev="Self-Issued Consulting (was at Microsoft)">Self-Issued Consulting</organization>
    </author>

    <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-federation-1_0.html">
        <front>
          <title>OpenID Federation 1.0</title>
		  <author fullname="R. Hedberg, Ed.">
            <organization>independent</organization>
          </author>
          <author fullname="Michael B. Jones">
            <organization>Self-Issued Consulting</organization>
          </author>
          <author fullname="A. Solberg">
            <organization>Sikt</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Yubico</organization>
          </author>
          <author fullname="Giuseppe De Marco">
            <organization>independent</organization>
          </author>
          <author fullname="Vladimir Dzhuvinov">
            <organization>Connect2id</organization>
          </author>
          <date day="15" month="October" year="2025"/>
        </front>
</reference>

# Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that MAY cover technology that MAY be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -00 

   *  Initial version
