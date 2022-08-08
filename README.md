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


## 1.1 Motivation

UCAN is transport agnostic, but was originally designed with the web in mind. This specification is the evolution of the original use case for UCAN: passing tokens around in headers. A major advantage of UCAN is content addressing gives automatic cachability, saving network bandwidth.

# 2 Headers

## 2.1 Entry Point

The entry-point for a UCAN chain MUST be `authorization: bearer <ucan>`. This is the only field REQUIRED to implement this spec.

## 2.2 UCAN header Table

Each row of a UCAN header table MUST use the key `ucan`, followed by the CID, a space (separator), and then the JWT-encoded UCAN.

``` abnf
ucan-header = "ucan: " <cid> " " <ucan-jwt>
```

``` javascript
// Example
[
  {"authorization": "bearer eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA"},
  {"ucan": "QmXiZ3sFXw811R8TrwaNeYvCF9Pv1nEmVpeEMEVpApzVhC eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtoS0paOVdvV1dnZVdqSnd3QU14VDh4c2tMelJzbURYSzZ1NktuVjlnR0pCViIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwibmJmIjo0ODA0MTQzNDEyLCJleHAiOjU0MzUyOTU0MTIsImF0dCI6W10sInByZiI6W119.u21cahr9wE_-KV_WHZmDRUlUGsMomc8jiDNwLYa-ETyJwCh8VtfPRSDwxNC3g2sv0hmqE9_467idq_T4wnLdBA"},
  {"ucan": "bafkreidrgwjljxy6s7o5uvrifxnweffgi7chmye3pn6wyisv2n4b3uordi eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJpc3MiOiJkaWQ6a2V5Ono2TWtxbmJOaTl2ZHRENERLUWhySDJZR1d0Qmd3QjNuNDEyQVFUOExnUjdBNjdFRyIsImF1ZCI6ImRpZDprZXk6ejZNa2ZndFhrQ25iOUxYbjhCbnlqeFJNbkt0RmdaYzc0TTY4NzN2NjFxQ2NLSGprIiwiZXhwIjo0ODA0MTQzNDEyLCJhdHQiOltdLCJwcmYiOltdfQ.MAntHVdUqeW97v4EPrSJjZ0P9GcLLFhFIdEYEHAdmv4x2CDfntUaqDzAgMCxwKCNBCAXBFvy1AT15ZFHs022AQ"}
]
```

# 3 Cache Signalling

The server MAY respond with 

# 4 Continuation

It is possible that more UCAN segments are needed to complete validation.

The server MUST respond with an `HTTP 403 Forbidden` status. FIXME use nonstandard HTTP status? (Others have for token use cases, e.g. 499, but could conflict with anoth nonstanard use)

# 5 Errors

## 5.1 Invalid UCAN

The server MUST respond with an `HTTP 401 Unauthorized` in response to a complete-but-invalid.

# XX. Acknowledgments

# XX. FAQ

## XX.1 Should the headers have an `X-` prefix?

The `X-` prefix was deprecated in [RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)
