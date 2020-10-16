# Tips & fine-tunings

From time to time it makes sense to analyse the log entries to
optimise the refresh runs. For example, by time you may end up adding
keys to the key ring that are not published at all.
gpg-key-ring-refresher will discover these as they weren't
on your initially configured preskiplist.

Also, the information provided by the key servers should be the same
no matter what key server happens to be the source. In practise, this
never holds and you'll notice some key servers may have little or no
information related to the keys in your key ring.

## The PGP keys that are not found

Assuming you've set the log level of gpg-key-ring-refresher to
INFO or more talkative, the log will contain a list of keys that were
not found during a refresh run. If keys continuously appear as "not found"
in the log, then they probably haven't been published at all and should
be put on the preskiplist. Doing this shortens the overall execution times
of gpg-key-ring-refresher (and means less pain to the key servers).

For example, run
```
user@host:~$ perl -n -e 'print "$1\n" if (/^.+\sKey\s([a-f\d]{16})\s\(status.+not\sfound\)$/i)' < .gpg-key-ring-refresher.log | sort | uniq -d
```
to get a list of keys that repeatedly show up as not found. For a digestible
version with occurrence counts of the former and out of love for one-liners,
run:
```
user@host:~$ perl -n -e 'print "$1\n" if (/^.+\sKey\s([a-f\d]{16})\s\(status.+not\sfound\)$/i)' < .gpg-key-ring-refresher.log | sort | perl -n -e 'if (/^([a-f\d]{16})$/i) { if ($key ne "$1") { print "$key:$count\n"; $key = $1; $count = 1; } else { $key = $1; $count++; } } END { print "$key:$count\n"; }' | tail -n +2

1111222233334444:12
AAAAAAAAAAAAAAAA:3
AAAABBBBCCCCDDDD:12
BBBBBBBBBBBBBBBB:8
...
```
## The choice of key servers

Again, assuming the log level is INFO or chattier, gpg-key-ring-refresher
writes a summary of the key info returned by each key server. If a server
continuously appear as returning no info on keys or info of very few keys, it
may make sense to remove the key server altogether from the configured
key server list. Prior to removal, however, check whether
info on those "very few" keys is provided by the remaining key servers.

Below is an example on how `keys.openpgp.org` starts to stand out as a
candidate for removal after only four rounds worth of gpg-key-ring-refresher
runs:
```
user@host:~$ grep 'provided info ' .gpg-key-ring-refresher.log | perl -p -e 's/^.+\s(\S+)\sprovided\sinfo\son\s(\d+)\skeys.+$/$1:$2/g' | sort
keyserver.ubuntu.com:19
keyserver.ubuntu.com:20
keyserver.ubuntu.com:20
keyserver.ubuntu.com:28
keys.openpgp.org:0
keys.openpgp.org:0
keys.openpgp.org:0
keys.openpgp.org:1
pgp.circl.lu:20
pgp.circl.lu:20
pgp.circl.lu:23
pgp.circl.lu:39
pgp.key-server.io:0
pgp.key-server.io:15
pgp.key-server.io:20
pgp.key-server.io:22
pgp.surfnet.nl:0
pgp.surfnet.nl:20
pgp.surfnet.nl:20
pgp.surfnet.nl:42
```
