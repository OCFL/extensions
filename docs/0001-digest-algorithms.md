# OCFL Community Extension 0001: Digest Algorithms

  * Authors: OCFL Editors
  * Minimum OCFL Version: 1.0
  * Obsoletes: n/a
  * Obsoleted by: n/a

## Overview

This extension is an index of additional digest algorithms that are defined in community extensions. It provides a controlled vocabulary of digest algorithm names that may be used to indicate the given algorithm in `fixity` blocks of OCFL Objects, and links their defining extensions.

## Digest Algorithms Defined in Community Extensions

--------------------------------
| Digest Algorithm Name | Note |
--------------------------------
| blake2b-160           | BLAKE2 digest using the 2B variant (64 bit) with size 160 bits as defined by [RFC7693](https://tools.ietf.org/html/rfc7693). MUST be encoded using hex (base16) encoding [RFC4648](https://tools.ietf.org/html/rfc4648). For example, the <code>blake2b-160</code> digest of a zero-length bitstream is `3345524abf6bbe1809449224b5972c41790b6cf2` (40 hex digits long). |
| blake2b-256           | BLAKE2 digest using the 2B variant (64 bit) with size 256 bits as defined by [RFC7693](https://tools.ietf.org/html/rfc7693). MUST be encoded using hex (base16) encoding [RFC4648](https://tools.ietf.org/html/rfc4648). For example, the <code>blake2b-256</code> digest of a zero-length bitstream starts as follows `0e5751c026e543b2e8ab2eb06099daa1d1e5df47...` (64 hex digits long). |
| blake2b-384           | BLAKE2 digest using the 2B variant (64 bit) with size 384 bits as defined by [RFC7693](https://tools.ietf.org/html/rfc7693). MUST be encoded using hex (base16) encoding [RFC4648](https://tools.ietf.org/html/rfc4648). For example, the <code>blake2b-384</code> digest of a zero-length bitstream starts as follows `b32811423377f52d7862286ee1a72ee540524380...` (96 hex digits long). |
| sha512/256            | SHA-512 algorithm with 256 output as defined by [FIPS-180-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf). MUST be encoded using hex (base16) encoding [RFC4648](https://tools.ietf.org/html/rfc4648). For example, the sha512 digest of a zero-length bitstream starts `c672b8d1ef56ed28ab87c3622c5114069bdd3ad7...` (64 hex digits long). |
--------------------------------

## Maintenance

In order to have an additional digest algorithm listed here, please submit a pull request on this extension that adds it to the table. New entries should have a name that does not conflict with those defined in the OCFL Specification or this community extension, and is preferably in common use for the given algorithm. In the case that long description is required it may be appropriate to submit a new extension describing the algorithm along with an update to this extension that links to it.
