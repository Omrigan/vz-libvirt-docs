========
virtlxcd
========

------------------------------
libvirt LXC management daemon
------------------------------

:Manual section: 8
:Manual group: Virtualization Support

.. contents::

SYNOPSIS
========

``virtlxcd`` [*OPTION*]...


DESCRIPTION
===========

The ``virtlxcd`` program is a server side daemon component of the libvirt
virtualization management system.

It is one of a collection of modular daemons that replace functionality
previously provided by the monolithic ``libvirtd`` daemon.

This daemon runs on virtualization hosts to provide management for LXC
containers.

The ``virtlxcd`` daemon only listens for requests on a local Unix domain
socket. Remote off-host access and backwards compatibility with legacy
clients expecting ``libvirtd`` is provided by the ``virtproxy`` daemon.

Restarting ``virtlxcd`` does not interrupt running guests. Guests continue to
operate and changes in their state will generally be picked up automatically
during startup. None the less it is recommended to avoid restarting with
running guests whenever practical.


SYSTEM SOCKET ACTIVATION
========================

The ``virtlxcd`` daemon is capable of starting in two modes.

In the traditional mode, it will create and listen on UNIX sockets itself.

In socket activation mode, it will rely on systemd to create and listen
on the UNIX sockets and pass them as pre-opened file descriptors. In this
mode most of the socket related config options in
``/etc/libvirt/virtlxcd.conf`` will no longer have any effect.

Socket activation mode is generally the default when running on a host
OS that uses systemd. To revert to the traditional mode, all the socket
unit files must be masked:

::

   $ systemctl mask virtlxcd.socket virtlxcd-ro.socket \
      virtlxcd-admin.socket

If using libvirt-guests service then the ordering for that service needs to be
adapted so that it is ordered after the service unit instead of the socket unit.
Since dependencies and ordering cannot be changed with drop-in overrides, the
whole libvirt-guests unit file needs to be changed.  In order to preserve such
change copy the installed ``/usr/lib/systemd/system/libvirt-guests.service`` to
``/etc/systemd/system/libvirt-guests.service`` and make the change there,
specifically make sure the ``After=`` ordering mentions ``virtlxcd.service`` and
not ``virtlxcd.socket``:

::

   [Unit]
   After=virtlxcd.service


OPTIONS
=======

``-h``, ``--help``

Display command line help usage then exit.

``-d``, ``--daemon``

Run as a daemon & write PID file.

``-f``, ``--config *FILE*``

Use this configuration file, overriding the default value.

``-p``, ``--pid-file *FILE*``

Use this name for the PID file, overriding the default value.

``-t``, ``--timeout *SECONDS*``

Exit after timeout period (in seconds), provided there are neither any client
connections nor any running domains.

``-v``, ``--verbose``

Enable output of verbose messages.

``--version``

Display version information then exit.


SIGNALS
=======

On receipt of ``SIGHUP`` ``virtlxcd`` will reload its configuration.


FILES
=====

The ``virtlxcd`` program must be ran as root. Trying to start the program under
a different user results in error.

* ``/etc/libvirt/virtlxcd.conf``

The default configuration file used by ``virtlxcd``, unless overridden on the
command line using the ``-f`` | ``--config`` option.

In addition to the default configuration file, ``virtlxcd`` reads
configuration for the LXC driver from:

* ``/etc/libvirt/lxc.conf``

This file contains various knobs and default values for virtual machines
created within LXC driver, and offers a way to override the built in defaults,
Location of this file can't be overridden by any command line switch.

* ``/run/libvirt/virtlxcd-sock``
* ``/run/libvirt/virtlxcd-sock-ro``
* ``/run/libvirt/virtlxcd-admin-sock``

The sockets ``virtlxcd`` will use.

The TLS **Server** private key ``virtlxcd`` will use.

* ``/run/virtlxcd.pid``

The PID file to use, unless overridden by the ``-p`` | ``--pid-file`` option.


EXAMPLES
========

To retrieve the version of ``virtlxcd``:

::

  # virtlxcd --version
  virtlxcd (libvirt) 8.5.0


To start ``virtlxcd``, instructing it to daemonize and create a PID file:

::

  # virtlxcd -d
  # ls -la /run/virtlxcd.pid
  -rw-r--r-- 1 root root 6 Jul  9 02:40 /run/virtlxcd.pid


BUGS
====

Please report all bugs you discover.  This should be done via either:

#. the mailing list

   `https://libvirt.org/contact.html <https://libvirt.org/contact.html>`_

#. the bug tracker

   `https://libvirt.org/bugs.html <https://libvirt.org/bugs.html>`_

Alternatively, you may report bugs to your software distributor / vendor.


AUTHORS
=======

Please refer to the AUTHORS file distributed with libvirt.


COPYRIGHT
=========

Copyright (C) 2006-2020 Red Hat, Inc., and the authors listed in the
libvirt AUTHORS file.


LICENSE
=======

``virtlxcd`` is distributed under the terms of the GNU LGPL v2.1+.
This is free software; see the source for copying conditions. There
is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE


SEE ALSO
========

virsh(1), libvirtd(8),
`https://www.libvirt.org/daemons.html <https://www.libvirt.org/daemons.html>`_,
`https://www.libvirt.org/drvlxc.html <https://www.libvirt.org/drvlxc.html>`_
