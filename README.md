# UCAN HTTP Header Specification v0.1.0

## Editors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)

## Authors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)
* [Daniel Holmgren](https://github.com/dholms), [Bluesky](https://blueskyweb.xyz/)

# 0. Abstract

[User-Controlled Authorization Network (UCAN)](https://github.com/ucan-wg/spec) is a trustless, secure, local-first, user-originated authorization and revocation scheme. This document describes a protocol for transmitting UCAN credential chains over HTTP headers, signalling that UCANs have been cached, requesting more UCANs, and failure states.

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 1 Introduction

The UCAN spec itself is transport agnostic. This specification describes how to transfer UCANs over HTTP headers.

## 1.1 Cache Advantages

The content addressing in the UCAN proofs (`prf`) field enforces immutibility of their content. This makes it possible to: 

1. Break up the serialization of collections of UCAN chains
2. Deduplicate UCAN proofs across requests

In avoiding transferring the token with all dependencies with every request, the sender gains bandwidth efficiency while still being able to validate UCAN proofs that it has received before (either directly or via gossip with other providers). 

This nessesitates additional steps in the protocol to handle requesting further UCAN proofs, and signalling that the UCANs in a request have been cached.

## 1.2 Bearer Tokens

A UCAN is not strictly a bearer token, since it MUST include the recipient in the `aud` field, and thus needs to be able to sign the token. The sender is more than a mere bearer.

A desirable feature of UCAN is that it is interpretable by systems that understand existing token formats based on JWT. For instance, most backend web frameworks come with an authorization plugin. These typically depend on auth tokens being provided as [`Authorization: Bearer`](https://datatracker.ietf.org/doc/html/rfc6750).

# 2 Request Headers

The HTTP headers MUST include an `Authorization: Bearer` header, and MAY include zero or more `ucan` fields that form a CID-to-UCAN table.

| Header                  | Type               | Description                 | Required | Cardinality  |
| ----------------------- | ------------------ | --------------------------- | -------- | ------------ |
| `Authorization: Bearer` | `<ucan_jwt>`       | Entrypoint "top-level" UCAN | Yes      | One          |
| `ucan`                  | `<cid> <ucan-jwt>` | Mapping of CID to UCAN      | No       | Zero or more |

## 2.1 Entry Point

The entry-point for a UCAN chain MUST be `Authorization: Bearer <ucan>`. This is the only REQUIRED field. The UCAN contained in this field MUST be [encoded as a JWT](https://www.rfc-editor.org/rfc/rfc7519#section-3). Per the UCAN spec, this token SHOULD be unique per request.

``` abnf
ucan-entrypoint = "Authorization:" 1*SP "Bearer" 1*SP <ucan-jwt>
```

## 2.2 UCAN-by-CID Table

The entrypoint UCAN MAY contain CID references to further UCANs as "proofs" (values in the `prf` field). For the entrypoint UCAN to be valid, these MUST also be valid and available.

Since UCANs MAY be cached in previous requests, including UCANs in this table is OPTIONAL. There MAY be zero or more UCAN CID headers. Together, these form a UCAN-by-CID table.

### 2.2.1 Row

Each row of the UCAN CID MUST use the header field `ucan`, followed by the CID, a space (separator), and finally a JWT-encoded UCAN.

The `cid` portion of each `ucan` header MUST be the CID for the following JWT. The CID MUST be given in the same format as provided in one of the other UCAN's proof `prf` field.

Regardless the encoding type specified in by the CID multiformat, the JWT itself MUST be provided as a JWT. Hash validation of UCANs by the recipient is RECOMMENDED. Any non-raw CIDs MUST be deterministically encoded, and thus are able to be encoded as a JWT (per the UCAN spec). A validator is REQUIRED to re-encode the JWT UCAN in the target CID multiformat when validating its hash.

``` abnf
ucan-header = "ucan:" 1*SP <cid> 1*SP <ucan-jwt>
```

## 2.3 Example

``` javascript
[
  {"Authorization": "Bearer eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA"},
  {"ucan": "QmXiZ3sFXw811R8TrwaNeYvCF9Pv1nEmVpeEMEVpApzVhC eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtoS0paOVdvV1dnZVdqSnd3QU14VDh4c2tMelJzbURYSzZ1NktuVjlnR0pCViIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwibmJmIjo0ODA0MTQzNDEyLCJleHAiOjU0MzUyOTU0MTIsImF0dCI6W10sInByZiI6W119.u21cahr9wE_-KV_WHZmDRUlUGsMomc8jiDNwLYa-ETyJwCh8VtfPRSDwxNC3g2sv0hmqE9_467idq_T4wnLdBA"},
  {"ucan": "bafkreidrgwjljxy6s7o5uvrifxnweffgi7chmye3pn6wyisv2n4b3uordi eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtxbmJOaTl2ZHRENERLUWhySDJZR1d0Qmd3QjNuNDEyQVFUOExnUjdBNjdFRyIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwiZXhwIjo0ODA0MTQzNDEyLCJhdHQiOltdLCJwcmYiOltdfQ.MAntHVdUqeW97v4EPrSJjZ0P9GcLLFhFIdEYEHAdmv4x2CDfntUaqDzAgMCxwKCNBCAXBFvy1AT15ZFHs022AQ"}
]
```

# X Response

## X.1 Success

The general success case is left unspecified. The success status code MUST be set by the application, based on its internal semantics.

## X.2. Cache TTL

The responder MAY include a signal that it has cached the UCANs from the request via a TTL header. The header MUST have the field `ucan-cache-expiry` and value set to the expiry in [Unix time](https://en.wikipedia.org/wiki/Unix_time).

``` abnf
ucan-ttl-header = "ucan-cache-expiry:" 1*SP <utc-timestamp>
```

## C Errors

The errors are intended to remain as compatible with [RFC6750](https://www.rfc-editor.org/rfc/rfc6750.html#section-3.1) as possible.

## C.1 Complete-but-Invalid UCAN

If the UCAN provided can be checked, but is found to be expired, revoked, malformed, or otherwise invalid, the recipient MUST respond with an `HTTP 401 Unauthorized`. This case MAY be encountered even without the entire UCAN chain present, such as when no valid proof is listen in the proof chain.

## C.2 Insufficient Capability Scope

If the UCAN is does not include sufficient authority to perform the requestor's action, the recipient MUST respond with an `HTTP 403 Forbidden`. This case MAY be encountered without the entire chain present, even the outermost UCAN layer could claim insufficient authorty.

## C.3 Missing Dependencies

In the case where the recipient is missing some further proof or proofs in a UCAN chain, it MUST respond with an [`HTTP 510 Not Extended`](https://datatracker.ietf.org/doc/html/rfc2774#section-7). The `Cache-Control: max-age=<seconds>` header MUST be set. The UCANs that were received that generated this response MUST be considered cached until the `max-age` TTL expires.

The body of the response MUST include a JSON object with a `prf` field. The value of this field MUST be an array of the required UCAN CIDs.

``` javascript
{ "prf": ["QmXiZ3sFXw811R8TrwaNeYvCF9Pv1nEmVpeEMEVpApzVhC", "bafkreidrgwjljxy6s7o5uvrifxnweffgi7chmye3pn6wyisv2n4b3uordi"] }
```

# XX. Acknowledgments

Many thanks to the authors of [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750.html) — Michael B. Jones and Dick Hardt — for their work in defining the bearer authorization method.

Thank you to [Chris Joel](https://github.com/cdata) of [Subconcious](https://subconscious.substack.com/), and the [Bluesky](https://blueskyweb.xyz) and [Fission](https://fission.codes) teams for pioneering this format.

# XX. FAQ

## XX.1 Should the headers have an `X-` prefix?

The `X-` prefix was deprecated in [RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)
