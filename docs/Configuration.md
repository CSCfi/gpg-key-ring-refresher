# Configuration

The precedence of sources of configuration information is

1. command line parameters
1. configuration file
1. hard-coded defaults.

Hard-coded defaults act as fall-backs if no other source defines a
configuration parameter. You may check the configuration used with
```
    gpg-key-ring-refresher showconf
    gpg-key-ring-refresher -c mytailoured.conf showconf
```
## 1. Contents of the configuration file

The configuration file may define all or some of the following configuration
sections:

* `][] i_am_no_great_fan_of_ini_files ]][`
* `][] hmmm___some_key_servers_are_also_needed  ]][`
* `][] skip_these_gpg_keys ]][`
* `][] and_logging_to_a_file_would_be_handy ]][`
* `][] key_specific_signature_limits ]][`

All lines in the configuration file ahead of the first section are ignored.
The order of the sections is irrelevant.

### 1.1. Section `][] i_am_no_great_fan_of_ini_files ]][`

This admittedly non-intuitively named section is used for defining
some or all of the following parameter value pairs (defaults included):

The master key ring that is to be refreshed

`KEYRINGGNUPGHOME=/home/user/.gnupg`

The maximum number of times a key server can fail in a row until it gets
excluded as a source of key information
  
`MAX_NUMBER_OF_CONSECUTIVELY_FAILED_REFRESH_ATTEMPTS_PER_KEY_SERVER=10`

The maximum number of seconds we wait until we consider the refresh
attempt as failed or hanging due to no or a partial response from a
key server
  
`MAX_WAIT_REFRESH_REQUEST_SECONDS=180`

The number of keys the script tries to refresh at every refresh request to
a key server

`NUM_KEYS_TO_REFRESH_PER_REQUEST=15`

The stem of the temporary key ring directories under which info from
the refreshes is stored
  
`TEMPKEYRINGTOPDIRSTEM=/tmp/gpg-key-ring-refresher`

The amount of seconds we wait between refresh attempts to key servers

`WAIT_GIVEN_SECONDS_BETWEEN_REFRESH_REQUESTS=180`

A maximum random amount of extra seconds we wait between refresh
attempts to key servers. This is summed up with the earlier
parameter. Be nice with the key servers and don't clog them with
requests in rapid succession, otherwise the key servers will end up
dropping your requests or not returning any data.

`WAIT_MAX_RANDOM_SECONDS_BETWEEN_REFRESH_REQUESTS=180`

The maximum number of signatures permitted for a key by default,
of course, YMMV (or, now that we are all so brexcited, a more
appropriate expression in directively orthodox Brusselian EU speak
would be VKPV). See, section `key_specific_signature_limits` below
for overriding.

`MAX_NUMBER_OF_SIGNATURES_PER_KEY=1000`

The maximum number of new signatures permitted for a key by default
during a single refresh run. See, section `key_specific_signature_limits`
below for overriding.

`MAX_NUMBER_OF_NEW_SIGNATURES_PER_KEY=100`

### 1.2. Section `][] hmmm___some_key_servers_are_also_needed  ]][`

Here you can define the key servers that will be tried for key information.
Define one key server per line. E.g.
```
pgp.circl.lu
pgp.surfnet.nl
cool.forgedkeys.gov
```
### 1.3. Section `][] skip_these_gpg_keys ]][`

A list (one per line) of PGP keys you don't want to be refreshed. A typical
entry on this list is a key that hasn't been published on any key server.
The identifiers are expected to be 16 hex digits long, e.g.
```
0x1111222233334444
5555666677778888
```
Revoked and expired keys are filtered out dynamically before the refresh
attempts begin.

### 1.4. Section `][] and_logging_to_a_file_would_be_handy ]][`

This section defines the Log::Log4perl initialization parameters. All
lines here get concatenated into the FILELOGGERCONF configuration
parameter and this value then fed to the Log::Log4perl
initializer. E.g.
```
log4perl.rootLogger=INFO, LOGFILE
log4perl.appender.LOGFILE=Log::Log4perl::Appender::File
log4perl.appender.LOGFILE.filename=sub { "$ENV{HOME}/.gpg-key-ring-refresher.log"; }
log4perl.appender.LOGFILE.mode=append
log4perl.appender.LOGFILE.layout=PatternLayout
log4perl.appender.LOGFILE.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%p] - %m%n
```
### 1.5. Section `][] key_specific_signature_limits ]][`

In this section you may specify key specific signature limits. The limits
given here override the global defaults MAX_NUMBER_OF_SIGNATURES_PER_KEY
and MAX_NUMBER_OF_NEW_SIGNATURES_PER_KEY.

The entries are tuples made of the key identifier, the maximum number of
signatures permitted for the key and the maximum number of new signatures
permitted on a single refresh run, all separated by whitespace e.g.
```
AAAABBBBCCCCDDDD 2000 50
1111111122222222 140 20
```

If you set either number to zero, it means there is no upper limit on
the number of signatures (max or per refresh run). You should _never_
set both to zero since this effectively leaves the key vulnerable to a
signature flooding attack.
