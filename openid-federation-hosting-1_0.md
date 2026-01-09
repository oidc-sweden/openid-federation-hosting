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

This specification defines an extension to OpenID Federation that enables Subordinate Statements to include a Claim that indicates where a subject’s Entity Configuration is located. The extension introduces the `ec_location` Claim, which specifies an alternative location for retrieving the subject’s Entity Configuration. This functionality allows Entity Configuration data to be hosted at an Intermediate Entity or at another location, which can be essential for enabling legacy systems or entities that do not have full federation support to register and operate within a federation.

{mainmatter}

# Introduction

OpenID Federation 1.0 [@!OpenID.Federation] defines a configuration endpoint where Federation Entities should publish their Entity Configuration. The URL for this endpoint is the Entity Identifier URL concatenated with `/.well-known/openid-federation`.

One situation where a Federation Entity may not be able to make their Entity Configuration available at this URL is when their Entity Configuration is hosted by a Superior Entity and published under a URL based on a domain name that is owned by the Superior Entity. Reasons for this may be that an Entity registering to a federation is not fully OpenID Federation-compatible, or simply that it cannot expose its Entity Configuration at this URL.

Another situation is when a Federation Entity, for legacy or other reasons, has an identifier that is not a URL. This is a case that is valid according for several Entity Types that may be supported by OpenID Federation.

Not publishing the Entity Configuration at the configuration endpoint is a choice by the Federation Entity itself with the clear consequence that the Federation Entity will not be discoverable based on their Entity Identifier alone.

This specification defines the `ec_location` Claim to be used in Subordinate Statements, holding a location where the Entity Configuration of the subordinate can be obtained.

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in BCP 14 [@!RFC2119]
[@!RFC8174] when, and only when, they appear in all capitals, as shown here.

## Terminology

This specification uses the terms "Claim" and "JSON Web Token (JWT)" as defined by JSON Web Token (JWT) [@!RFC7519], and "Entity", "Entity Identifier", "Trust Anchor", "Federation Entity", "Entity Statement", "Entity Configuration", "Subordinate Statement", "Intermediate Entity", "Leaf Entity", "Subordinate Entity", "Superior Entity", and "Trust Chain" defined in OpenID Federation 1.0 [@!OpenID.Federation].

# The ec\_location Claim {#the_ec_location_claim}

This section defines the `ec_location` Claim that MAY be included in a Subordinate Statement.

Using this Claim, an alternative location for the subject’s Entity Configuration can be specified in a Subordinate Statement. Typically, this location differs from the configuration endpoint defined in Section 9 of [@!OpenID.Federation].

The value of the `ec_location` Claim is a single string that specifies the URL where the subject’s Entity Configuration is available. This URL MUST use either the https scheme or the data URL scheme [@!RFC2397], containing the complete Entity Configuration as a signed JWT.

A data URL MUST use the media type `application/entity-statement+jwt` and MUST NOT include a `;base64` declaration. The Entity Configuration data MUST be provided using compact serialization without additional Base64 encoding.

Note: Base64 encoding is omitted because the Entity Configuration JWT already uses a URL-safe format.

Section 9 of [@!OpenID.Federation] mandates that Intermediate Entities and Trust Anchors publish their Entity Configuration at the configuration endpoint. Therefore, it is RECOMMENDED to use the `ec_location` Claim in Subordinate Statements only for Leaf Entities.

An Intermediate Entity creating a Subordinate Statement that includes the `ec_location` Claim SHOULD NOT mark this Claim as critical (see Section 13.4 of [@!OpenID.Federation]). The reason is that Trust Chain validation does not depend on support for `ec_location`. If a peer supplies the full chain, it can be validated even when the Claim is not understood.

```json=
{
  "iss" : "https://superior-entity.example.com",
  "sub" : "urn:example:legacy:rp",
  ...  
  "ec_location" : "https://superior-entity.example.com/hosted/ec/example-legacy-rp",
  ...
}
```

**Example 1**: Excerpt of the Subordinate Statement issued by `https://superior-entity.example.com` for the subject `urn:example:legacy:rp`, where the Intermediate Entity itself hosts the subject's Entity Configuration. This is indicated using the `ec_location` Claim.

```json=
  "ec_location" : "data:application/entity-statement+jwt,eyJhb...Qssw5c"
```

**Example 2**: Using a data URL as the value for `ec_location`.

# Trust Chain Construction {#trust_chain_construction}

Section 10.1 of [@!OpenID.Federation] describes the process of establishing a Trust Chain using a bottom-up approach. In this approach, the process starts with the Entity Configuration of the Entity whose metadata and Trust Chain are to be resolved, and then constructs a chain up to a trusted Trust Anchor. This process uses the `authority_hints` Claim of each fetched Entity Statement to identify the next Superior Entity. The procedure is repeated until at least one trusted Trust Anchor is reached, for which a complete chain of statements can be evaluated.

Using this process will not work unless a Leaf Entity publishes its Entity Configuration at the configuration endpoint, as specified in Section 9 of [@!OpenID.Federation]. Not all Entities within a federation may be able to adhere to this requirement. Therefore, alternative chain construction mechanisms need to be used.

Section 17.2.2 of [@!OpenID.Federation] defines a top-down discovery pattern, where a Trust Chain is built starting from a known Trust Anchor.

The most important difference between top-down and bottom-up chain construction, as defined in Section 10.1 of [@!OpenID.Federation], is the order in which Entity Statements are collected. In top-down construction, the Subordinate Statement of an Entity is fetched before the Entity Configuration for the same subject. This fact, together with the use of the `ec_location` Claim ((#the_ec_location_claim)), enables the construction of a chain even when a subject Entity does not publish its Entity Configuration at the configuration endpoint.

The top-down chain construction algorithm SHOULD be used in federations where Entity Configurations may be hosted at locations other than an Entity’s configuration endpoint.

Federations that rely on the use of Resolvers (see Section 10.6 of [@!OpenID.Federation]) implementing top-down chain construction can use this capability to enable hosting of Entity Configuration documents at more convenient locations, while still ensuring the availability of validated peer Entity data via resolve endpoints.

# Security Considerations

## Ensuring Entity Identifier Uniqueness

An Entity that exposes its Entity Configuration at the configuration endpoint, as specified in Section 9 of [@!OpenID.Federation], will implicitly prove its ownership of the domain on which this document is published. This ensures that the same Entity Identifier cannot be used by separate Entities within the federation.

However, if an Entity cannot publish its Entity Configuration at this location and instead relies on its Superior Entity to include the `ec_location` Claim in the Subordinate Statement for the Entity, there is no implicit proof of ownership of the domain. If this is not combined with naming restrictions, this may lead to a federation where several separate Entities have the same Entity Identifier, which in turn may lead to risks concerning Entity impersonation.

Suppose the following federation setup:

- Under the Intermediate Entity IE-1, there is an Entity named `https://op.example.com`. This Entity publishes its Entity Configuration at its configuration endpoint.

- Intermediate Entity IE-2, which supports hosting of Entity Configurations on behalf of Subordinate Entities, provides a Subordinate Statement for an Entity that is also named `https://op.example.com`. This Subordinate Statement includes the `ec_location` Claim pointing to the hosted Entity Configuration location. 

~~~ ascii-art
                      +-------------+
                      |     TA      |
                      +-------------+
                          |     |
                __________|     |__________
               |                           |
        +------v------+             +------v------+
        |    IE-1     |             |     IE-2    |
        +-------------+             +--------+----+   +........................+
               |                             |________| https://op.example.com |
   +-----------v------------+                         +........................+
   | https://op.example.com |                                   Hosted
   +------------------------+
~~~

**Figure 1**: Illustration of how an Entity may impersonate another legitimate Entity if no security restrictions are applied within the federation.

This is an example of how the integrity of the federation may be violated if no restrictions regarding Entity Identifier assignments exist. In the example above, the Subordinate Entity under the Intermediate Entity IE-2 manages to get IE-2 to issue a Subordinate Statement for an Entity with an Entity Identifier that is already held by a legitimate Entity.

A federation that supports the use of the `ec_location` Claim MUST therefore define rules for Intermediate Entities issuing Subordinate Statements containing the `ec_location` Claim. Before an Intermediate Entity creates a Subordinate Statement for an Entity that does not publish its Entity Configuration at the configuration endpoint, the following restrictions MUST be fulfilled:

- If the Entity Identifier for the Subordinate Entity is a URL, the Intermediate Entity MUST ensure that the actor requesting a Subordinate Statement owns, or has rights to use, the domain given by the Entity Identifier URL.

- Regardless of whether the Entity Identifier for the Subordinate Entity is a URL or not, the Intermediate Entity MUST ensure that this identifier is not already assigned to another Entity within the federation.

How the above requirements are implemented is out of scope for this specification.

Furthermore, a Trust Anchor or Intermediate Entity MAY define Naming Constraints, as specified in Section 6.2.2 of [@!OpenID.Federation], to further ensure Entity Identifier integrity.

# Acknowledgments

We would like to thank the following individuals for their comments, ideas, and contributions to this implementation profile and to the initial set of implementations.

- Leif Johansson, Siros
- Felix Hellman, Helagon

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
