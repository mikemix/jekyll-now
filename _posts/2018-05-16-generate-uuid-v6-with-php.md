---
layout: post
title: Generate UUID v6 in PHP
---

The [RFC 4122](https://tools.ietf.org/html/rfc4122) describes five versions of UUID,
but none of them is optimized to be used as a primary key in a relational database
we are so fond of.

Brad Peabody [proposed Version 6](https://bradleypeabody.github.io/uuidv6/):

> TL;DR: 'Version 6' UUIDs have the date/time encoded from high to low bytes
(bit-shifting around the version field in order to preserve its location) 
and thus sort correctly by time when treated as an opaque bunch of bytes; 
the clock sequence can be used to avoid duplicates generated at the same time; 
it's okay to use random data from a good PRNG in place of the MAC address; 
the rest is the same as RFC 4122.

Pity I didn't found any PHP implementation, needed to write one myself. 
Go and grab it from Packagist as [mikemix/php-uuid-v6](https://packagist.org/packages/mikemix/php-uuid-v6).
Hope it helps someone!
