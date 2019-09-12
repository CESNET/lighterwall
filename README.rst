========================================================
LighterWall - simple IPv4/IPv6 agnostic iptables wrapper
========================================================

* Author: `Pavel Kácha`_
* Copyright: 2019 `Pavel Kácha`_
* License: MIT

LighterWall is a wrapper to ease writing of corresponding IPv4/IPv6
firewalls, and while at it, creating groups of IP addresses (or other
identifiers), for which ip[6]?tables will be run multiple times.

.. _`Pavel Kácha`: ph@cesnet.cz

Files
=====

lighterwall
  Static script. Place it (or link to it) to some suitable autorun place,
  like /etc/network/if-pre-up.d in Debian, or to initscript/unit.

lighterwall-rules
  Place this into /etc where it will be picked up by lighterwall script.

Usage
=====

::

  lighterwall start|stop|debug [rules file]

Cpt Obvious. Optionally you can use different rules file as a second
argument. That also means you can debug rules from different file, and if
you cut the branch underneath you, you can reboot and lighterwall picks up
original rules.

See also lighterwall-rules.example.

Macros
======

lighterwall-rules can contain a couple of macros.

dict
----

::

  dict _name_  str4  str6

If you use _name_ on the ipt command line, both iptables and ip6tables
will be run for respective address. You can use _OMIT_ instead of IP to
signal that this _name_ does not have corresponding IP.

It can be used also for non address strings, like icmp-host-prohibited vs
icmp6-adm-prohibited.

iplist
------

::

  iplist _name_ IP IP IP IP ...

Addresses are the mix of IPv4 and IPv6 addresses. Script recognizes which is
which and runs iptables and ip6tables respectively.

list
----

::

  list _name_ str str str str

Same as iplist, but without ip address cleverness. You can use it for naming
the ports, for which ip[6]?tables should run repeatedly, regardless the
protocol.

sub/endsub
----------

Where you would have to create the list of already defined placeholders, you
can use sub ... endsub::

  list    _admins_
  sub
      iplist  _bofh_      192.0.2.100  192.0.2.105  2001:DB8::2:100  2001:DB8::2:105
      iplist  _pfy_       192.0.2.173  2001:DB8::2:173
  endsub

This creates _bofh_ and _pfy_ placeholders and also adds them into _admins_
list in one step. It is nestable.

Rules
=====

lighterwall-rules must contain rules() shell functions, which contains
iptables-like lines (but starting with 'ipt' pseudocommand), but with
possible additional macro placeholders, like::

  ipt --append INPUT --protocol tcp --dport 22 --syn --source _admins_ --jump ACCEPT

Note that for more substitutions on one line ipt unrolls M x N times (where
it makes sense, it does not combine IPv4 defined placeholders with IPv6
ones).

Copyright
=========
Distribution is governed by MIT License, see LICENSE file.
