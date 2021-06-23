.. 
   Copyright (C) Internet Systems Consortium, Inc. ("ISC")
   
   This Source Code Form is subject to the terms of the Mozilla Public
   License, v. 2.0. If a copy of the MPL was not distributed with this
   file, you can obtain one at https://mozilla.org/MPL/2.0/.
   
   See the COPYRIGHT file distributed with this work for additional
   information regarding copyright ownership.

Notes for BIND 9.17.7
---------------------

New Features
~~~~~~~~~~~~

- Support for DNS over TLS (DoT) has been added: the ``dig`` tool is now
  able to send DoT queries (``+tls`` option) and ``named`` can handle
  DoT queries (``listen-on tls ...`` option). ``named`` can use either a
  certificate provided by the user or an ephemeral certificate generated
  automatically upon startup. :gl:`#1840`

- A new configuration option, ``stale-refresh-time``, has been
  introduced. It allows a stale RRset to be served directly from cache
  for a period of time after a failed lookup, before a new attempt to
  refresh it is made. :gl:`#2066`

Feature Changes
~~~~~~~~~~~~~~~

- The ``dig``, ``host``, and ``nslookup`` tools have been converted to
  use the new network manager API rather than the older ISC socket API.

  As a side effect of this change, the ``dig +unexpected`` option no
  longer works. This could previously be used to diagnose broken servers
  or network configurations by listening for replies from servers other
  than the one that was queried. With the new API, such answers are
  filtered before they ever reach ``dig``, so the option has been
  removed. :gl:`#2140`

- The network manager API is now used by ``named`` to send zone transfer
  requests. :gl:`#2016`

Bug Fixes
~~~~~~~~~

- ``named`` could crash with an assertion failure if a TCP connection
  were closed while a request was still being processed. :gl:`#2227`

- ``named`` acting as a resolver could incorrectly treat signed zones
  with no DS record at the parent as bogus. Such zones should be treated
  as insecure. This has been fixed. :gl:`#2236`

- After a Negative Trust Anchor (NTA) is added, BIND performs periodic
  checks to see if it is still necessary. If BIND encountered a failure
  while creating a query to perform such a check, it attempted to
  dereference a ``NULL`` pointer, resulting in a crash. :gl:`#2244`

- A problem obtaining glue records could prevent a stub zone from
  functioning properly, if the authoritative server for the zone were
  configured for minimal responses. :gl:`#1736`

- ``UV_EOF`` is no longer treated as a ``TCP4RecvErr`` or a
  ``TCP6RecvErr``. :gl:`#2208`
