# UCAN as Bearer Token Specification v0.3.0

## Editors

* [Brooklyn Zelenka], [Fission]

## Authors

* [Brooklyn Zelenka], [Fission]
* [Daniel Holmgren], [Bluesky]
* [Hugo Dias], [DAG House]
* [Quinn Wilton], [Fission]

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14] when, and only when, they appear in all capitals, as shown here.

# 0 Abstract

[User-Controlled Authorization Network (UCAN)][UCAN] is a trustless, secure, local-first, user-originated authorization and revocation scheme. This document describes a protocol for transmitting UCAN credential chains over HTTP headers, signalling that UCANs have been cached, requesting more UCANs, and failure states.

# 1 Introduction

The UCAN spec itself is transport agnostic. This specification describes how to transfer [UCAN]s as headers in an HTTP request.

## 1.1 Cache Advantages

The content addressing in the UCAN proofs (`prf`) field enforces immutability of their content. This makes it possible to: 

1. Break up the serialization of collections of UCAN chains
2. Deduplicate UCAN proofs across requests

In avoiding transferring the token with all dependencies with every request, the sender gains bandwidth efficiency while still being able to validate UCAN proofs that it has received before (either directly or via gossip with other providers). 

This necessitates additional steps in the protocol to handle requesting further UCAN proofs, and signalling that the UCANs in a request have been cached.

## 1.2 Bearer Tokens

A UCAN is not strictly a bearer token, since it MUST include the recipient in the `aud` field, and thus needs to be able to sign the token. The sender is more than a mere bearer.

A desirable feature of UCAN is that it is interpretable by systems that understand existing token formats based on JWT. For instance, most backend web frameworks come with an authorization plugin. These typically depend on auth tokens being provided as [`Authorization: Bearer`][RFC 6750].

# 2 Request Headers

The HTTP headers MUST include an `Authorization: Bearer` header, and MAY include zero or one `ucans` field that contains a UCAN array.

| Header                  | Type         | Description                  | Required |
| ----------------------- | ------------ | ---------------------------- | -------- |
| `Authorization: Bearer` | `ucan-jwt`   | Entry point "top-level" UCAN | Yes      |
| `ucans`                 | `[ucan-jwt]` | UCAN proof array             | No       |

## 2.1 Entry Point

The entry-point for a UCAN chain MUST be `Authorization: Bearer <ucan>`. This is the only REQUIRED field. The UCAN contained in this field MUST be encoded as a [JWT]. Per the UCAN spec, this token SHOULD be unique per request.

``` abnf
ucan-entry-point = "Authorization:" 1*SP "Bearer" 1*SP <ucan-jwt>
```

## 2.2 UCAN Proof Array

The entry point UCAN MAY contain CID references to further UCANs as "proofs" (values in the `prf` field). For the entry point UCAN to be valid, these MUST also be valid and available. Since UCANs MAY be cached in previous requests, including UCAN mappings in this table is OPTIONAL. Only one `ucans` header SHOULD be used. More than one `ucans` header MUST NOT be used.

``` abnf
ucan-header = "ucan:" 1*SP <ucan-list>
ucan-list = <ucan-jwt> *("," *SP <ucan-jwt>) 
```

Note that this field MUST be comma separated, but MUST NOT be enclosed in brackets.

Each entry in the UCAN proof array MUST be presented as one or more JWT-formatted UCANs separated by commas. To recover the CID of each entry, the [canonical CID] format SHOULD be tried by default. If a UCAN lists a different CID format for a proof, then the elements array MUST be rehashed with the relevant CID configuration.

## 2.3 Example

``` javascript
[
  {"Authorization": "Bearer eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA"},
  {"ucans": "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtoS0paOVdvV1dnZVdqSnd3QU14VDh4c2tMelJzbURYSzZ1NktuVjlnR0pCViIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwibmJmIjo0ODA0MTQzNDEyLCJleHAiOjU0MzUyOTU0MTIsImF0dCI6W10sInByZiI6W119.u21cahr9wE_-KV_WHZmDRUlUGsMomc8jiDNwLYa-ETyJwCh8VtfPRSDwxNC3g2sv0hmqE9_467idq_T4wnLdBA", "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtxbmJOaTl2ZHRENERLUWhySDJZR1d0Qmd3QjNuNDEyQVFUOExnUjdBNjdFRyIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwiZXhwIjo0ODA0MTQzNDEyLCJhdHQiOltdLCJwcmYiOltdfQ.MAntHVdUqeW97v4EPrSJjZ0P9GcLLFhFIdEYEHAdmv4x2CDfntUaqDzAgMCxwKCNBCAXBFvy1AT15ZFHs022AQ"}
]
```

# 3 Response

## 3.1 Success

The general success case is left unspecified. The success status code MUST be set by the application, based on its internal semantics.

## 3.2 Cache and Expiry

The responder MAY include a signal that it has cached the UCANs from the request via a TTL header. The header MUST have the field `ucan-cache-expiry` and value set to the expiry in [Unix time].

``` abnf
ucan-ttl-header = "ucan-cache-expiry:" 1*SP <utc-timestamp>
```

## 3.3 Errors

The errors are intended to remain as compatible with [RFC6750] as possible.

### 3.3.1 Complete-but-Invalid UCAN

If the UCAN provided can be checked, but is found to be expired, revoked, malformed, or otherwise invalid, the recipient MUST respond with an `HTTP 401 Unauthorized`. This case MAY be encountered even without the entire UCAN chain present, such as when no valid proof is listen in the proof chain.

### 3.3.2 Insufficient Capability Scope

If the UCAN is does not include sufficient authority to perform the requestor's action, the recipient MUST respond with an `HTTP 403 Forbidden`. This case MAY be encountered without the entire chain present, even the outermost UCAN layer could claim insufficient authority.

### 3.3.3 Missing Proofs

In the case where the recipient is missing some further proof or proofs in a UCAN chain, it MUST respond with an [`HTTP 510 Not Extended`]. The [`ucan-cache-expiry` header] MUST be set. The UCANs that generated this response MUST be considered cached until either the cache expires or they are explicitly requested in a further "missing proof" response.

The body of the response MUST include a JSON object with a `prf` field. The value of this field MUST be an array of the required UCAN CIDs.

#### 3.3.3.1 Response Body Example

``` javascript
{ "prf": ["QmXiZ3sFXw811R8TrwaNeYvCF9Pv1nEmVpeEMEVpApzVhC", "bafkreidrgwjljxy6s7o5uvrifxnweffgi7chmye3pn6wyisv2n4b3uordi"] }
```

# 4 FAQ

## 4.1 Why disallow duplicate headers?

Duplicate headers are not handled consistently by all clients. Restricting to a single field is the simplest cross-client solution to this, as long as the spec is followed.

## 4.2 Why not include the UCAN CIDs?

Several attacks are possible if UCANs aren't validated. If CIDs aren't validated, at least two attacks are posisble: [privilege escelation] and [cache poisoning], as UCAN delegation proofs depends on a correct hash-linked structure. Many implementations cache a list of UCANs that they've already validated — avoiding cache poisoning is thus very important.

By not including the CID in the header table, the recipient is forced to hash (and thus validate) the CIDs for each entry. If presented with a claimed CID as a table, implementers could ignore CID validation, breaking a core part of the proof chain security model. Hash functions are very fast on a couple kilobytes of data (the limit of most browsers in 8KB). While this strategy may be abused to force more hashing, the overhead is still very low.

# 5 Acknowledgments

Many thanks to the authors of [RFC 6750] — Michael B. Jones and Dick Hardt — for their work in defining the bearer authorization method.

Thank you to [Chris Joel] of [Subconscious], and the [Bluesky] and [Fission] teams for pioneering this format.

<!-- Internal Links -->

[`ucan-cache-expiry` header]: #32-cache-and-expiry

<!-- External Links -->

[BCP 14]: https://www.rfc-editor.org/info/bcp14
[Bluesky]: https://blueskyweb.xyz/
[Brooklyn Zelenka]: https://github.com/expede
[Chris Joel]: https://github.com/cdata 
[DAG House]: https://dag.house
[Daniel Holmgren]: https://github.com/dholms
[Fission]: https://fission.codes
[Hugo Dias]: https://github.com/hugomrdias
[JWT]: https://www.rfc-editor.org/rfc/rfc7519#section-3
[Quinn Wilton]: https://github.com/QuinnWilton
[RFC 6750]: https://www.rfc-editor.org/rfc/rfc6750.html
[Subconscious]: https://subconscious.substack.com
[UCAN]: https://github.com/ucan-wg/spec
[Unix time]: https://en.wikipedia.org/wiki/Unix_time
[`HTTP 510 Not Extended`]: https://datatracker.ietf.org/doc/html/rfc2774#section-7
[cache poisoning]: https://en.wikipedia.org/wiki/Cache_poisoning
[privilede escelation]: https://en.wikipedia.org/wiki/Privilege_escalation
