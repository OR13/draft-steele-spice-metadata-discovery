---
title: "SPICE Metadata Discovery"
category: info

docname: draft-steele-spice-metadata-discovery-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Secure Patterns for Internet CrEdentials"
keyword:
 - credential
 - presentation
 - identity
 - metadata
venue:
  group: "Secure Patterns for Internet CrEdentials"
  type: "Working Group"
  mail: "spice@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spice/"
  github: "OR13/draft-steele-spice-metadata-discovery"
  latest: "https://OR13.github.io/draft-steele-spice-metadata-discovery/draft-steele-spice-metadata-discovery.html"

author:
 -
    fullname: "Orie Steele"
    organization: Transmute
    email: "orie@transmute.industries"

normative:


informative:
  I-D.draft-ietf-ace-key-groupcomm: ACE-KEY-GROUPCOMM
  I-D.draft-ietf-oauth-sd-jwt-vc: SD-JWT-VC
  BoundedContexts: https://martinfowler.com/bliki/BoundedContext.html
  RFC8949: CBOR
  RFC8259: JSON
  RFC9518:
  RFC7071:
  RFC6839:
  RFC8725: JWT-BCP
  JSON-LD:
    title: A JSON-based Serialization for Linked Data
    target: https://www.w3.org/TR/json-ld11/
  DIDs:
    title: Decentralized Identifiers (DIDs) v1.0
    target: https://www.w3.org/TR/did-core/
  DIDsFO:
    title: DID 1.0 Formal Objections Report
    target: https://www.w3.org/2022/03/did-fo-report.html
  VCs:
    title: Securing Verifiable Credentials using JOSE and COSE
    target: https://www.w3.org/TR/vc-jose-cose/
  OpenIDConnect:
    title: Open ID Connect
    target: https://openid.net/specs/openid-connect-core-1_0.html


--- abstract

Entities interested in digital credentials need to express and discover preferences for working with them.

Before issuance, holders need to discover what credentials are supported, and issuers need to discover if a holder's wallet is safe enough to store their credentials.

Before presentation, holder's need to discovery verifier's encryption keys, and which presentation formats a verifier supports.

After presentation, verifiers need to discover any new key material or status changes related to credentials.

This document enables issuers, holders and verifiers to discover supported protocols and formats for keys, claims, credentials and proofs.

--- middle

# Introduction

The abstractions and relationships that have evolved to support digital credentials and secure systems are challenging to comprehend and used in conflciting and contradictory ways in different organizations, and security specifications.

An identity (also known an entity), is identified (distinguished) through identifiers (names), and can fulfill multiple roles, including being an Issuer, Holder, Relying Party or Verifier of digital credentials and presentations.

The attributes (claims, or reputation statements), capabilities, and relations associated with an identifier are expressed as metadata regarding the identifier.

Because it remains easier to rediscover the fundamentals of digital identity and credentials than it is to read or comprehend the previous work, very similar, yet unfortunately incompatible systems are continiously created, and through their adoption the digital identity ecosystem becomes increasingly difficult to comprehend or support.

This document is not meant to solve all the challenges facing organizations seeking to adopt digital identity and credentialing technology.
However, this document attempts to describe an identifier and metadata architecture, that reflects the current state of the art, and addresses the challenge of fragmentation and data siloing, through a distillation of the essentials needed to support digital credentials.

# Terminology

{::boilerplate bcp14-tagged}

# Background

In Open ID Connect {{OpenIDConnect}}, and {{-SD-JWT-VC}}, identifiers are URLs, metadata is discovered through URLs, claims are expressed in JSON, {{-JSON}}, and content is described using content types.

Request:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

GET /.well-known/openid-configuration HTTP/1.1
Host: issuer.example
Accept: \
  application/json;charset=utf-8, \
  application/json
~~~

Response:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

HTTP/1.1 200 Ok
Content-Type: \
  application/json

{
  "issuer": "https://issuer.example",
  "token_endpoint": "https://issuer.example/api/oauth/token",
  "jwks_uri": "https://issuer.example/.well-known/jwks.json",
  "response_types_supported": [
    "token"
  ],
  "claims_supported": [
    "aud",
    "exp",
    "iat",
    "iss",
    "sub"
  ],
  "id_token_signing_alg_values_supported": [
    "ES384"
  ],
  "token_endpoint_auth_signing_alg_values_supported": [
    "ES384"
  ],
  "id_token_encrypted_response_alg": [
    "ECDH-ES+A256KW
  ],
  "id_token_encrypted_response_enc": [
    "A128CBC-HS256"
  ]
  ...
}
~~~

In {{DIDs}} and {{VCs}} the identifiers are URLs, metadata is discovered through dereferencing URLs, claims are expressed in {{JSON-LD}}, and content is described using content types.

For example:


Request:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

GET /identifiers/did:example:123 HTTP/1.1
Host: resolver.example
Accept: \
  application/did+json
~~~

Response:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

HTTP/1.1 200 Ok
Content-Type: \
  application/json

{
  "id": "did:example:123"
}
~~~


In {{-ACE-KEY-GROUPCOMM}} the identifiers are URLs, metadata discovered through dereferencing URLs, attributes are expressed in CBOR, {{-CBOR}}, and content is described using content types.

For example:

Request:

~~~
Header: POST (Code=0.02)
Uri-Host: "kdc.example.com"
Uri-Path: "ace-group"
Uri-Path: "g1"
Content-Format: "application/ace-groupcomm+cbor"
Payload (in CBOR diagnostic notation,
         with AUTH_CRED and POP_EVIDENCE being CBOR byte strings):
  { "scope": << [ "group1", ["sender", "receiver"] ] >> ,
    "get_creds": [true, ["sender"], []], "client_cred": AUTH_CRED,
    "cnonce": h'25a8991cd700ac01', "client_cred_verify": POP_EVIDENCE }
~~~

Response:

~~~
Header: Created (Code=2.01)
Content-Format: "application/ace-groupcomm+cbor"
Location-Path: "kdc.example.com"
Location-Path: "g1"
Location-Path: "nodes"
Location-Path: "c101"
Payload (in CBOR diagnostic notation,
         with KEY being a CBOR byte strings):
  { "gkty": 13, "key": KEY, "num": 12, "exp": 1609459200,
    "creds": [ AUTH_CRED_1, AUTH_CRED_2 ],
    "peer_roles": ["sender", ["sender", "receiver"]],
    "peer_identifiers": [ ID1, ID2 ] }
~~~

These examples highlight perhaps the only common attributes of modern digital credential systems: structured identifiers (URLs and URNs) and content types.

## The layering problem

Effective standards limit optionality, improve interoperability, and connect bounded contexts, {{BoundedContexts}} through interfaces that require trivial to acquire and well understood tooling.

With respect to structured identifiers and content types, the simplest solutions select a single identifier type, and a single content type.

For example, a hypothetical AcmeHyperProtocol might rely only on URLs and JSON.

Support for AcmeHyperProtocol would be easy, developers would need only to have reliable libraries for working with URLs and JSON.

Software does not evolve this way.

It is common to see dependencies that solve several seperate problems efficiently, bundled together, such that it is impossible to take a dependency on just the part that is needed.

This problem is then reflected in standards, because effective standards describe effective software systems.

In OpenID Connect, we see URLs, but we also see URNs; we see `application/json`, but we also see `application/jwt` nested inside of it.

In DIDs and VCs, we see URLs, but we also see DIDs (URNs); we see `application/json`, but we also see `application/ld+json` nested inside of it, even further, we see `application/n-quads` constraining what `application/json` can express through the use of `application/ld+json`. We see URLs contraining what DIDs can express, through reuse of the path, query and fragment. We see a desire to wrap easily understood content types such as `application/jwk+json` or `application/jwt` in less easy to understand JSON-LD content types. Where does this nesting come from?

{{-RFC9518}} explains in Section 4.7: "standards efforts should focus on providing concrete utility to the majority of their users as published, rather than being a "framework" where interoperability is not immediately available. Internet functions should not make every aspect of their operation extensible; boundaries between modules should be designed in a way that allows evolution, while still offering meaningful functionality."

In order to enable consumers leverage their prefered identifiers and content types, some specifications take a "big tent" approach, created an open ended extensibility mechansism, and then providing a single mandatory to implement instantiation of it. In the worst cases, {{DIDsFO}}, standards insist on providing the estensiblity mechanisms, and refuse to provide mandatory to implement instances, and through doing so, ensure no interoperability is achievable without a second document, enabling a profile that is actually usable. It could be argued that OAuth has similar deficiencies, and that OpenID Connect solved this same problem through a suite of secondary documents.

It might seem impossible to support extensibility and interoperability simultaneously, but as the authors on {{-RFC9518}} and Martin Fowler in {{BoundedContexts}} points out, the key to succeeding is ensuring the layering is correct.

# Strategy & Tactics

Editors note: This whole section is far too ranty... the objective is to provide generic guidance to address the problem, before providing concrete recommendations and a single set of mandatory to implement APIs.

Be honest about the hierarchical model.

Digital identity solutions always assume supremacy of certain building blocks.

Open ID Connect assumes HTTP, JSON and JWT.

DIDs and VCs assume URNs and JSON-LD.

CBOR is frequently assumed for new work, due to its similarity to JSON and superior storage and transmission costs.

It is better to build a very useful, and easy to implement specification that solves a problem, than it is to build a hard to use, and hard to implement specification that seeks inclusivity at the cost of efficiency, interoperability and simplicity.

Once the essential components, and their relative priorities (read importance or rank) are established, assemble them such that each might be replaced in the future, for example HTTP might be replaced with some other transport, JSON with CBOR, JWT with CWT, etc.

It is ok, if replacing a single component, requires replacing a set of connected sub components, for example moving from JSON to CBOR, might also require moving from JWT to CWT.

While it might seem valuable to support multiple formats, it is one of the highest cost decisions a standard can fail to make for implementers, and it should be avoided at all costs.

# Structured Identifiers

## Compound Identifiers

Compound identifiers are commonly used to address the challend of expressing relations between distinct identifiers.

In URLS for http resources, depending on the content type, the compount identifier `https://issuer.example/capabilities#f81d4fae-7dec-11d0-a765-00a0c91e6bf6` might express that "f81d4fae-7dec-11d0-a765-00a0c91e6bf6" is a sub resource of "https://issuer.example/capabilities" which is a sub resource of "https://issuer.example".

When constructing compoind identifiers, it is important to consider how negotiation is impacted by leveraging different parts of a structured identifier.

For example, a server can assist with negotiation for response types for `https://issuer.example/capabilities`, but sub resources identified by a fragment have their response type controlled by their parent resource.

## Global Uniqueness

Global uniqueness is a deseriable property of structed identifiers.

Ensuring global uniquess often tends towards centralization or federation, because some system or entity must ensure that distinct identities are not able to contest, or claim a single unique identifier.

One way to achieve global uniqueness is to produce a compound identifier where the authority or structure of the identifier establishes the uniquenss through relation to itself.

For example `https://issuer.example/f81d4fae-7dec-11d0-a765-00a0c91e6bf6` relies on the authority `https://issuer.example` to maintain the linkage between resource identifier and resource representations.

Another example `https://issuer.example/urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6` uses a URN namespace to establish a globally unique identifier `urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6` and then uses `https://issuer.example` to establish a compound identifier which is globally unique.

## Sharing Structured Identifiers

Some systems might wish to share structured identifiers, while maintaining authority for representations.

For example `https://issuer.example/urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6` and `https://verifier.example/urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6` might both use `urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6` to express a globally unique identifier for an attribute key or attribute value.

When independent entities share globally unique identifiers, they are responsible for ensuring that the identifier is used consistently.

For example:

`https://resolver1.example/identifiers/did:example:123` could serve completely different JSON-LD for `did:example:123` than `https://resolver2.example/identifiers/did:example:123`.


Because unlike URLs, URNs are not resolvable by themselves, trust in resources represented with URNs comes from the system you are communicating with, and unlike URLs, there is no commonly understood scheme for communiation, such as http encoded in the identifier.

# Content Types

Type systems such as the Hindley–Milner type system, can provide much stronger security properties than well defined content types, but both serve a similar purpose, which is to express the intended processing and capabilies of data structures.

## Structured Suffixes

{{-RFC7071}} defined a JSON based media type for expressing reputation, `application/reputon+json`.

The `+json` part is a structured suffix as described in {{-RFC6839}}.

Defining a new media type with a structured suffix, allows for systems that support content type negotiation to respond with more precise content types.

This property of responding with less ambigious content types is part of secure system design, and {{-JWT-BCP}} notes that using more specific types can help protect against certain attacks.

Using a more specific content type comes at the cost of being "generally understood", for example `application/json` is often expected or hard coded in software, and returning `application/reputon+json` could cause software to fail higher up in the application stack.

## Allowing Additional Properties

It is common for content types to rely on map or object datastructures for their top level serializations.

This is because a MAP is easily extended with new key value pairs, which when not understood can be ignored, whereas a string or array, might cause processing errors if extended.

`application/jwk-set+json` builds on `application/json` and expresses a set of cryptographic keys represented by values, that are consistent with `application/jwk+json`.

In cases where additional properties can be present, implementations should take advantage of this and avoid creating new media types, until the number of new properties is substantial enough to justify ensuring security and processing considerations specific to the new type cause faults.

For example, consider the following URL and resource expressing "keys that are use to make attribute assertions about a subject".


Request:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

GET /keys/assertion HTTP/1.1
Host: issuer.example
Accept: \
  application/jwk-set+json
~~~

Response:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

HTTP/1.1 200 Ok
Content-Type: \
  application/json

{
  "keys": [
    {
      "kid": "urn:ietf:params:oauth:jwk-thumbprint:sha-256:NzbLsXh8...sRGC9Xs"
      "kty": "EC",
      "crv": "P-256",
      "alg": "ES256",
      "x": "MKBCTNIcKUSDii11ySs3526iDZ8AiTo7Tu6KPAqv7D4",
      "y": "4Etl6SRW2YiLUrN5vfvVHuhp7x8PxltmWWlbbM4IFyM"
    }
  ]
}
~~~

The `keys` property is part of `application/jwk-set+json`, and additional properties can be added at this layer, without creating a new media type, however, those properties will not be well understood in the context of `application/jwk-set+json`.

Similarly inside each key the `alg` property is optional, but when present it signals the algorithm the key is restricted to be used with.

Additional properties might be added to the keys themselves, however, those properties will not be well understood in the context of `application/jwk+json`.


# SPICE Metadata Discovery

Identifiers for issuer, holders and verifiers MUST be URLs.

For example:

~~~
https://issuer.example
~~~

The scheme of the URL MUST support resolving content types.

For example:

Request:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

GET / HTTP/1.1
Host: issuer.example
Accept: \
  application/json
~~~

Response:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

HTTP/1.1 200 Ok
Content-Type: \
  application/json

{
  "capabilities": "https://issuer.example/capabilities",
  ...
}
~~~

Content type specific sub resources MUST be expressved by reference, and MUST NOT be expressed by value.

For example:

Request:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

GET / HTTP/1.1
Host: issuer.example
Accept: \
  application/json
~~~

Response:

~~~ http-message
NOTE: '\' line wrapping per RFC 8792

HTTP/1.1 200 Ok
Content-Type: \
  application/json

{
  "jwks": "https://issuer.example/keys",
  ...
}
~~~

It is RECOMMENDED to avoid registering new media types, except in cases where a response might be confused for an existing media type in a way that impacts security.

It is RECOMMENDED to return content types that have message level integrity, to protect authenticity, and ensure transport agility is less impacted by transport specific security considerations.

It is RECOMMENDED to submit content types that have messsage level encryption, to protect confidentiality, and ensure transport agility is less impacted by transport specific security considerations.

It is RECOMMENDED that "purpose" be encoded in identifiers, not attributes of representations.

This allows for negotiation of different content types satisfying the same purpose.

Purpose is more than just "which algorithms", it is also the intended use of the algorithm.

For example, a key migth be used to sign assertions to create credentials, or challenges to enable authentication.


# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
