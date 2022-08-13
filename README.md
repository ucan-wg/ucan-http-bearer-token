# UCAN Over HTTP Headers Specification v0.1.0

## Editors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)

## Authors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)
* [Daniel Holmgren](https://github.com/dholms), [Bluesky](https://blueskyweb.xyz/)

# 0. Abstract

[User-Controlled Authorization Network (UCAN)](https://github.com/ucan-wg/spec) is a trustless, secure, local-first, user-originated authorization and revocation scheme. This document describes the protocol for transmitting UCAN credential chains over HTTP headers, signalling that some tokens have been cached, requesting more CIDs, and failure states.

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 1 Introduction

The UCAN spec itself is transport agnostic. This specification describes how to transfer UCANs over HTTP headers.

Content addressing in the [UCAN proofs (`prf`) field](FIXME link to UCAN spec) makes it possible to both: 

1. Break up the serialization of 
2. Deduplicate UCANs across requests

By not sending the entire token and all dependencies with every request, the sender gains bandwidth efficiency while still being able to validate tokens that depend on UCANs that it has received before (either directly or via gossip with other providers). However, this leads to additional complexities around potentially needing to request further UCAN tokens.





<!-- UCAN not strictly a bearer -->


A major advantage of UCAN is content addressing gives automatic cachability, saving network bandwidth.

<!-- FIXME consider also including responding with which capabilities the UCAN should provide -->

# 2 Headers

## 2.1 Entry Point

A desirable feature of UCAN is that it is interpretable by systems that understand existing token formats based on JWT. For instance, most backend web frameworks come with an authorization plugin. These typically depend on auth tokens being provided as [`Authorization: Bearer`](https://datatracker.ietf.org/doc/html/rfc6750).

The entry-point for a UCAN chain MUST be `Authorization: Bearer <ucan>`. This is the only REQUIRED field.

The UCAN contained in this field MUST be [encoded as a JWT](https://www.rfc-editor.org/rfc/rfc7519#section-3).

| Header                  | Type         | Description                 | Required | Unique |
| ----------------------- | ------------ | --------------------------- | -------- | ------ |
| `Authorization: Bearer` | `<ucan_jwt>` | Entrypoint "top-level" UCAN | Yes      | Yes    |

``` abnf
ucan-entrypoint = "Authorization:" 1*SP "Bearer" 1*SP <ucan-jwt>
```

## 2.2 UCAN CID Table

Each row of a UCAN header table MUST use the key `ucan`, followed by the CID, a space (separator), and finally the JWT-encoded UCAN. There MAY be zero or more UCAN CID headers. Together, these form a CID table.

| Header | Type               | Description                  | Required | Unique |
| ------ | ------------------ | ---------------------------- | -------- | ------ |
| `ucan` | `<cid> <ucan-jwt>` | Single mapping of CID to JWT | No       | No     |

The `cid` portion of each `ucan` header MUST be the CID for the following JWT, and MUST be given in the same format as provided in one of the other UCAN's proof `prf` field.

Regardless the encoding type specified in by the CID multiformat, the JWT itself MUST be provided as a JWT. Hash validation of UCANs by the recipient is RECOMMENDED. Any non-raw CIDs are thus REQUIRED to re-encode the UCAN in the target CID multiformat when validating its hash.

``` abnf
ucan-header = "ucan:" 1*SP <cid> 1*SP <ucan-jwt>
```

### 2.2.1 Example

``` javascript
[
  {"Authorization": "Bearer eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA"},
  {"ucan": "QmXiZ3sFXw811R8TrwaNeYvCF9Pv1nEmVpeEMEVpApzVhC eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtoS0paOVdvV1dnZVdqSnd3QU14VDh4c2tMelJzbURYSzZ1NktuVjlnR0pCViIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwibmJmIjo0ODA0MTQzNDEyLCJleHAiOjU0MzUyOTU0MTIsImF0dCI6W10sInByZiI6W119.u21cahr9wE_-KV_WHZmDRUlUGsMomc8jiDNwLYa-ETyJwCh8VtfPRSDwxNC3g2sv0hmqE9_467idq_T4wnLdBA"},
  {"ucan": "bafkreidrgwjljxy6s7o5uvrifxnweffgi7chmye3pn6wyisv2n4b3uordi eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtxbmJOaTl2ZHRENERLUWhySDJZR1d0Qmd3QjNuNDEyQVFUOExnUjdBNjdFRyIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwiZXhwIjo0ODA0MTQzNDEyLCJhdHQiOltdLCJwcmYiOltdfQ.MAntHVdUqeW97v4EPrSJjZ0P9GcLLFhFIdEYEHAdmv4x2CDfntUaqDzAgMCxwKCNBCAXBFvy1AT15ZFHs022AQ"}
]
```

# A. Cache Signalling

The server MAY respond with 

# B Continuation

It is possible that more UCAN segments are needed to complete validation.

The server MUST respond with an `HTTP 403 Forbidden` status. FIXME use nonstandard HTTP status? (Others have for token use cases, e.g. 499, but could conflict with anoth nonstanard use)

# C Errors

## C.1 Invalid UCAN

The server MUST respond with an `HTTP 401 Unauthorized` in response to a complete-but-invalid.

# XX. Acknowledgments

Many thanks to the authors of [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750.html) — Michael B. Jones and Dick Hardt — for their work in defining the bearer authorization method.

# XX. FAQ

## XX.1 Should the headers have an `X-` prefix?

The `X-` prefix was deprecated in [RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)
