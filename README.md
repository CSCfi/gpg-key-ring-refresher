GPG key ring refresher
======================

This script shouldn't exist. But it does in an attempt to work around
the following shortcomings of the plain gpg tools and in particular the
refresh functionality:

 * gpg refresh connections to PGP key servers may hang,
 * PGP key servers may return some or no info on keys being refreshed, even
   though they would have info available,
 * gpg refresh and gpg-agent are not able to handle gracefully
   PGP keys targeted by signature flooding attacks, see e.g.
   * https://lwn.net/Articles/792366/ , or, 
   * https://anarc.at/blog/2019-07-30-pgp-flooding-attacks/
   * a long term solution to flooding attacks would be giving only the owner
     of a key the authority to decide what signatures to the key if any
     are published to PGP key servers. This could be achieved e.g. by the owner
     signing and uploading the signatures received.
   
This script runs gpg refreshes under the hood and tries to make reasonable
decisions as to what to do with the data gpg refresh returns (or decides not
to return). It autonomously refreshes the key information in the
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
listed at the end of the refresh run (see the log generated). These
keys ought to be added to the preskip list to avoid costly polling of
servers. Alternatively, ask the owners of those keys to upload
them to well-known servers. Likewise, the keys that have exceeded
the number of signatures allowed are reported. The number of signature
limits are controled by the global configuration parameters
MAX_NUMBER_OF_SIGNATURES_PER_KEY and MAX_NUMBER_OF_NEW_SIGNATURES_PER_KEY.
Key specific limits can be set in the configuration file's section
`key_specific_signature_limits`.

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

CSC - IT Centre for Science Ltd, www.csc.fi, 2020-
