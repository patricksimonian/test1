.. 
   Copyright (C) Internet Systems Consortium, Inc. ("ISC")
   
   This Source Code Form is subject to the terms of the Mozilla Public
   License, v. 2.0. If a copy of the MPL was not distributed with this
   file, you can obtain one at https://mozilla.org/MPL/2.0/.
   
   See the COPYRIGHT file distributed with this work for additional
   information regarding copyright ownership.

Notes for BIND 9.17.10
----------------------

New Features
~~~~~~~~~~~~

- Support for DNS-over-HTTPS (DoH) was added to ``named``. Because of
  this, the ``nghttp2`` HTTP/2 library is now required for building the
  development branch of BIND 9. Both TLS-encrypted and unencrypted
  HTTP/2 connections are supported (the latter may be used to offload
  encryption to other software).

  Note that there is no client-side support for HTTPS as yet; this will
  be added to ``dig`` in a future release. :gl:`#1144`

- ``named`` now supports XFR-over-TLS (XoT) for incoming as well as
  outgoing zone transfers. Addresses in a ``primaries`` list can now be
  accompanied by an optional ``tls`` keyword, followed by either the
  name of a previously configured ``tls`` statement or ``ephemeral``.
  :gl:`#2392`

- A new option, ``stale-answer-client-timeout``, has been added to
  improve ``named``'s behavior with respect to serving stale data. The
  option defines the amount of time ``named`` waits before attempting to
  answer the query with a stale RRset from cache. If a stale answer is
  found, ``named`` continues the ongoing fetches, attempting to refresh
  the RRset in cache until the ``resolver-query-timeout`` interval is
  reached.

  The default value is ``1800`` (in milliseconds) and the maximum value
  is limited to ``resolver-query-timeout`` minus one second. A value of
  ``0`` causes any available cached RRset to immediately be returned
  while still triggering a refresh of the data in cache.

  This new behavior can be disabled by setting
  ``stale-answer-client-timeout`` to ``off`` or ``disabled``. The new
  option has no effect if ``stale-answer-enable`` is disabled.
  :gl:`#2247`

Removed Features
~~~~~~~~~~~~~~~~

- A number of non-working configuration options that had been marked as
  obsolete in previous releases have now been removed completely. Using
  any of the following options is now considered a configuration
  failure: ``acache-cleaning-interval``, ``acache-enable``,
  ``additional-from-auth``, ``additional-from-cache``,
  ``allow-v6-synthesis``, ``cleaning-interval``, ``dnssec-enable``,
  ``dnssec-lookaside``, ``filter-aaaa``, ``filter-aaaa-on-v4``,
  ``filter-aaaa-on-v6``, ``geoip-use-ecs``, ``lwres``,
  ``max-acache-size``, ``nosit-udp-size``, ``queryport-pool-ports``,
  ``queryport-pool-updateinterval``, ``request-sit``, ``sit-secret``,
  ``support-ixfr``, ``use-queryport-pool``, ``use-ixfr``. :gl:`#1086`

Feature Changes
~~~~~~~~~~~~~~~

- When serve-stale is enabled and stale data is available, ``named`` now
  returns stale answers upon encountering any unexpected error in the
  query resolution process. This may happen, for example, if the
  ``fetches-per-server`` or ``fetches-per-zone`` limits are reached. In
  this case, ``named`` attempts to answer DNS requests with stale data,
  but does not start the ``stale-refresh-time`` window. :gl:`#2434`

- The default value of ``max-stale-ttl`` has been changed from 12 hours
  to 1 day and the default value of ``stale-answer-ttl`` has been
  changed from 1 second to 30 seconds, following :rfc:`8767`
  recommendations. :gl:`#2248`

- The SONAMEs for BIND 9 libraries now include the current BIND 9
  version number, in an effort to tightly couple internal libraries with
  a specific release. This change makes the BIND 9 release process both
  simpler and more consistent while also unequivocally preventing BIND 9
  binaries from silently loading wrong versions of shared libraries (or
  multiple versions of the same shared library) at startup. :gl:`#2387`

- When ``check-names`` is in effect, A records below an ``_spf``,
  ``_spf_rate``, or ``_spf_verify`` label (which are employed by the
  ``exists`` SPF mechanism defined in :rfc:`7208` section 5.7/appendix
  D.1) are no longer reported as warnings/errors. :gl:`#2377`

Bug Fixes
~~~~~~~~~

- ``named`` failed to start when its configuration included a zone with
  a non-builtin ``allow-update`` ACL attached. :gl:`#2413`

- Previously, ``dnssec-keyfromlabel`` crashed when operating on an ECDSA
  key. This has been fixed. :gl:`#2178`

- KASP incorrectly set signature validity to the value of the DNSKEY
  signature validity. This has been fixed. :gl:`#2383`

- When migrating to KASP, BIND 9 considered keys with the ``Inactive``
  and/or ``Delete`` timing metadata to be possible active keys. This has
  been fixed. :gl:`#2406`

- Fix the "three is a crowd" key rollover bug in KASP. When keys rolled
  faster than the time required to finish the rollover procedure, the
  successor relation equation failed because it assumed only two keys
  were taking part in a rollover. This could lead to premature removal
  of predecessor keys. BIND 9 now implements a recursive successor
  relation, as described in the paper "Flexible and Robust Key Rollover"
  (Equation (2)). :gl:`#2375`

- Performance of the DNSSEC verification code (used by
  ``dnssec-signzone``, ``dnssec-verify``, and mirror zones) has been
  improved. :gl:`#2073`
