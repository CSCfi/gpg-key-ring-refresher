GPG key ring refresher
======================

Refreshing key info of rings with numerous keys tends to suffer from
refresh attempts left hanging and attempts being unanswered thus making a
complete refresh impossible. This is mainly due to the load imposed
on public key servers.

This script tries to autonomously refresh the key information in a
given, arbitrary sized public key ring using a configured set of key
servers as sources. It splits the key ring into smaller sets and
attempts to refresh each set independently. It assumes the key ring
resides in a file named pubring.kbx (GPGv2) and trust information is
in trustdb.gpg. On a successful refresh, these two files are
updated. The recommended way of invocation is with cron.

Key info retrieval from the key servers is opportunistic: the script
trusts the first key server returning an answer on the key, the
remaining servers are not tried. If a key is not known to a key
server, then the remaining servers are tried one by one until info is
found or no more servers can be tried. Keys unknown to all servers are
listed at the end of the refresh run (see the log file). These
keys ought to be added to the preskiplist to avoid costly polling of
servers. Alternatively, ask the owners of those keys to upload
them to well-known servers.

By default, gpg-key-ring-refresher waits between key refresh connections in order
to tread lightly with the key servers. These defaults can be adjusted
in the configuration file or by using command line options if it seems
key servers stop providing key info due to too frequent refresh
attempts.

Requirements
------------

gpg-key-ring-refresher requires the following software to be present:

 * perl 5 - a recent version as of this writing
 * Log::Log4perl - CPAN module
 * Digest::MD5 - CPAN module
 * gpg2 executables (gpg, gpg-agent, dirmngr)
 * cp and tty executables

Nitty-gritty details
--------------------

See documentation under the docs directory.

License
-------

GPLv3, see LICENSE file. For the required auxiliary software, see their
corresponding licenses.

Author
------

Mika Silander

Copyright
---------

CSC - IT Centre for Science Ltd, www.csc.fi, 2020
