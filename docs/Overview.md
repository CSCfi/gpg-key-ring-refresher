## 1. Installation

Install `bin/gpg-key-ring-refresher` e.g. under `/usr/local/bin` to
offer it to all users of the (Linux) host. For individual use only,
`$HOME/bin` is fine.

Make sure the script has appropriate exec rights set.

## 2. Configuration

Unattended execution under cron is the recommended way to use the script.
E.g. edit the appropriate crontab file to contain a line like:
```
15 2 * * 6 test -x /home/user/bin/gpg-key-ring-refresher && /home/user/bin/gpg-key-ring-refresher -c /home/user/.gpg-key-ring-refresher.conf refresh
```
or, if you are happy with the hard-coded defaults:
```
15 2 * * 6 test -x /home/user/bin/gpg-key-ring-refresher && /home/user/bin/gpg-key-ring-refresher -H refresh
```
The above examples run the refresher every Saturday morning at 2:15. Raising
the frequency is not much point since PGP key servers then start to drop
refresh requests or return no info.

For the individual configuration parameters, see docs/Configuration.md.

## 3. Monitoring for or alerting on errors

Monitoring or alerting on errors is not supported by the
script itself. Instead you may use some monitoring system, e.g. Icinga,
Zabbix etc to check the log file gpg-key-ring-refresher
writes. The last line of a refresh run in the log tells the status
with which the run finished, e.g. your monitoring should at a minimum
look for non-zero status values from those lines:
```
2020-09-19 04:54:15 [INFO] Finished gpg-key-ring-refresher run, Sat Sep 19 04:54:15 2020 (status 0, duration 02:39:14)
```
Alternatively, you may write a separate cron script that alerts you by e.g.
email if such lines are found in the log.

In case a non-zero status is encountered, check the preceding log
lines for hints as to what went wrong. You may also raise the
verbosity of logging by setting the log level to ERROR or DEBUG if the
aforementioned hints are not informative enough.

## 4. Safeguards implemented

At startup gpg-key-ring-refresher attempts to create a pid file
in the master key ring directory to be refreshed. If a pid file already exists,
it bails out with a warning if the process owning the pid file is still
running. If not, it attempts to delete and recreate the pid file and lock
it exclusively.

MD5 sums are counted over the two files, pubring.kbx and trustdb.gpg,
that will be updated in a refresh. Locking is not attempted since most
other unrelated processes do not honour advisory file locks. If new
key data is found and the MD5 sums counted at the end of the refresh
run match the ones counted at the beginning, then the master key ring
files are updated. In case of a mismatch it means the master key ring
has been modified during the refresh run and we can't overwrite the
refreshed info over the key ring files without losing the other
changes. A following refresh run will correct the situation and
(re)fetch the updated key information.

## 5. Error codes

... some day some year maybe ... they should be quite self-explanatory though.
