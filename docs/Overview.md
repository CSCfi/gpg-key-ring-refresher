## 1. Installation

Install `bin/gpg-key-ring-refresher` e.g. under `/usr/local/bin` to
offer it to all users of the (Linux) host. For individual use only,
`$HOME/bin` is fine.

Make sure the script has appropriate exec rights set.

## 2. Usage

To get started, run `gpg-key-ring-refresher -h`:
```

A programme for refreshing a GPG2 key ring against info from PGP key servers.

    gpg-key-ring-refresher { -c conf_file_name | -H } [ -g gnupghomedir |
		  -w seconds | -s seconds | -r max_seconds |
		  -k number | -f number ] refresh
    gpg-key-ring-refresher -h
    gpg-key-ring-refresher { [ -c conf_file_name ] showconf | help }

    Options:
      -c = use given configuration file
      -H = use hard-coded configuration only
      -g = GnuPG home under which pubring.kbx and trustdb.gpg files are
           expected to be found. This is the key ring to be refreshed
           (conf option KEYRINGGNUPGHOME)
      -w = wait upto given number of seconds before assuming the refresh
           request to a PGP key server has failed (default is 180 seconds
           and this is the allowed minimum wait time, conf option
           MAX_WAIT_REFRESH_REQUEST_SECONDS)
      -s = sleep given amount of seconds between refresh tries (default 180,
           conf option WAIT_GIVEN_SECONDS_BETWEEN_REFRESH_REQUESTS)
      -r = sleep max random seconds between refresh tries (default 120,
           conf option WAIT_MAX_RANDOM_SECONDS_BETWEEN_REFRESH_REQUESTS)

           NB: wait times given with options -s and -r are summed up.
           These are intended to prevent swamping the pgp key servers
           with requests.

      -k = number of keys to refresh per refresh request (default 20,
           conf option NUM_KEYS_TO_REFRESH_PER_REQUEST)
      -f = max number of consecutively failed refreshes per key server
           (default 10, conf option
            MAX_NUMBER_OF_CONSECUTIVELY_FAILED_REFRESH_ATTEMPTS_PER_KEY_SERVER)
      -h = this help

    Commands:
      showconf  - show configuration parameters and values used	
      refresh   - refresh the key ring (requires either option -c or -H)
      help      - this help (same as option h)

    v. 0.0.5

```
## 3. Configuration

Unattended execution under cron is the recommended way to use the script.
E.g. add a line like the following to an appropriate crontab file:
```
15 2 * * 6 test -x /home/user/bin/gpg-key-ring-refresher && /home/user/bin/gpg-key-ring-refresher -c /home/user/.gpg-key-ring-refresher.conf refresh
```
or, if you are happy with the hard-coded defaults:
```
15 2 * * 6 test -x /home/user/bin/gpg-key-ring-refresher && /home/user/bin/gpg-key-ring-refresher -H refresh
```
The above examples run the refresher every Saturday morning at 2:15. Raising
the frequency is not much point since PGP key servers then start dropping
refresh requests or claiming they have no info.

For the individual configuration parameters, see docs/Configuration.md.

## 4. Monitoring for or alerting on errors

Monitoring or alerting on errors is not supported by the script
itself. Instead you may use some monitoring system, e.g. Icinga,
Zabbix etc to check the log entries of gpg-key-ring-refresher,
or, write a separate monitoring script yourself.
Assuming the logging level is set to INFO (or more detailed), then the last
line of a gpg-key-ring-refresher run in the log tells the exit
status of the run, e.g. your monitor should at a minimum
look for non-zero status values from those lines:
```
2020-09-19 04:54:15 [INFO] Finished gpg-key-ring-refresher run, Sat Sep 19 04:54:15 2020 (status 0, duration 02:39:14)
```
In case a non-zero status is encountered, check the preceding log
entries for hints as to what went wrong. You may also raise the
verbosity of logging by setting the log level to DEBUG, TRACE or ALL if the
aforementioned hints are not informative enough.

NB: Setting a more reticent log level than INFO will effectively leave
the log file with no indication of gpg-key-ring-refresher's
exit statuses.

## 5. Safeguards implemented

At startup, gpg-key-ring-refresher attempts to create a
pid file in the master key ring directory to be refreshed. If a pid
file already exists, we check whether the process owning the pid file
is still running. If it is running, we bail out with a warning, if not, we
attempt to delete and recreate the pid file and lock it exclusively.

To ensure integrity, MD5 sums are counted over pubring.kbx and
trustdb.gpg. Locking them is not attempted since most other unrelated
processes do not honour advisory file locks. If new key data is found
and the MD5 sums counted at the end of the refresh run match the ones
counted at the beginning, then the master key ring files are
updated. A mismatch means the master key ring has been modified during
the refresh run and we can't overwrite the refreshed info over the key
ring files without losing the other changes. Subsequent
gpg-key-ring-refresher runs will however mend the
situation as they eventually (re)fetch the same key information updates.

## 6. Error codes

... some day some year maybe ... they should be quite self-explanatory though.
