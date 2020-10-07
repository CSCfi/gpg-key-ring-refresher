Configuration
=============

The precedence of sources of configuration information is

    * command line parameters
    * configuration file
    * hard-coded defaults.

Hard-coded defaults act as fall-backs if no other source defines a
configuration parameter. You may check them with

    gpg-key-ring-refresher showconf

1. Contents of the configuration file
-------------------------------------

The configuration file may define all or some of the following configuration
sections:

* ][] i_am_no_great_fan_of_ini_files ]][
* ][] hmmm___some_key_servers_are_also_needed  ]][
* ][] skip_these_gpg_keys ]][
* ][] and_logging_to_a_file_would_be_handy ]][

All lines in the configuration file ahead of the first section are ignored.
The order of the sections is not relevant.

1.1. Section "][] i_am_no_great_fan_of_ini_files ]]["
-----------------------------------------------------

This section is used for defining parameter value pairs, namely the
following ones (defaults included):

The master key ring that is to be refreshed

KEYRINGGNUPGHOME=/home/user/.gnupg

The maximum number of times a key server can fail in a row until it gets
excluded as a source of key information
  
MAX_NUMBER_OF_CONSECUTIVELY_FAILED_REFRESH_ATTEMPTS_PER_KEY_SERVER=10

The maximum number of seconds we wait until we consider the refresh
attempt as failed or hanging due to no or a partial response from a
key server
  
MAX_WAIT_REFRESH_REQUEST_SECONDS=180

The number of keys the script tries to refresh at every refresh request to
a key server

NUM_KEYS_TO_REFRESH_PER_REQUEST=15

The stem of the temporary key ring directories under which the refresh
operations are performed
  
TEMPKEYRINGTOPDIRSTEM=/tmp/gpg-key-ring-refresher

The amount of seconds we wait between refresh attempts to key servers

WAIT_GIVEN_SECONDS_BETWEEN_REFRESH_REQUESTS=180

A maximum random amount of extra seconds we wait between refresh
attempts to key servers. This is summed up with the earlier
parameter. Be nice with the key servers and don't clog them with
requests in rapid succession. If not, the key servers will end up
dropping your requests or not returning any data.

WAIT_MAX_RANDOM_SECONDS_BETWEEN_REFRESH_REQUESTS=180

1.2. Section "][] hmmm___some_key_servers_are_also_needed  ]]["
---------------------------------------------------------------

Here you can define the key servers that will be tried for key information.
Define one key server per line.

1.3. Section "][] skip_these_gpg_keys ]]["
------------------------------------------

A list (one per line) of PGP keys you don't want to be refreshed. A typical
entry on this list is a key that hasn't been published on any key server.
The identifiers are expected to be 16 hex digits long, e.g.

0x1111222233334444
5555666677778888

1.4. Section "][] and_logging_to_a_file_would_be_handy ]]["
-----------------------------------------------------------

This section defines the Log::Log4perl initialization parameters. All
lines here get concatenated into the FILELOGGERCONF configuration
parameter and this value then fed to the Log::Log4perl
initializer. E.g.

log4perl.rootLogger=INFO, LOGFILE
log4perl.appender.LOGFILE=Log::Log4perl::Appender::File
log4perl.appender.LOGFILE.filename=sub { "/home/user/.gpg-key-ring-refresher.log"; }
log4perl.appender.LOGFILE.mode=append
log4perl.appender.LOGFILE.layout=PatternLayout
log4perl.appender.LOGFILE.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%p] - %m%n
