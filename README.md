# UCAN Over HTTP Headers Specification v0.1.0

## Editors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)

## Authors

* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)
* [Daniel Holmgren](https://github.com/dholms), [Bluesky](https://blueskyweb.xyz/)

# 0. Abstract

[User-Controlled Authorization Network (UCAN)](https://github.com/ucan-wg/spec) is a trustless, secure, local-first, user-originated authorization and revocation scheme. This document describes the protocol for transmitting UCAN credential chains over HTTP headers, signalling that some tokens have been cached, and failure states.

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 1. Introduction

## 1.1 Motivation


## Collection Format

A UCAN collection transmitted as HTTP headers MUST be formatted as follows:

Header key: UCAN [yes, we're using duplicates]
Header value: {CID}: {base64url encoded UCAN}

# XX. Acknowledgments

# XX. FAQ

## XX.1 Should the headers have an `X-` prefix?

The `X-` prefix was deprecated in [RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)
