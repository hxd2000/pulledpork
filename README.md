PulledPork
==========

PulledPork for Snort and Suricata rule management (from Google code)

Find us on Freenode (IRC) [`#ppork`](https://webchat.freenode.net/?channels=ppork)

Copyright (C) 2009-2020 JJ Cummings, Michael Shirk and the PulledPork Team!

Thank you for choosing to use PulledPork!  This file provides some basic
guidance on the usage of PulledPork.  Please be sure to read this file
thoroughly so that you don't overlook something!


## Features and Capabilities

 * Automated downloading, parsing, state modification and rule modification
   for all of your snort rulesets.
 * Checksum verification for all major rule downloads
 * Automatic generation of updated sid-msg.map file
 * Capability to include your local.rules in sid-msg.map file
 * Capability to pull rules tarballs from custom urls
 * Complete Shared Object support
 * Complete IP Reputation List support
 * Capability to download multiple disparate rulesets at once
 * Maintains accurate changelog
 * Capability to HUP processes after rules download and process
 * Aids in tuning of rulesets
 * Verbose output so that you know EXACTLY what is happening
 * Minimal Perl Module dependencies
 * Support for Suricata, and ETOpen/ETPro rulesets
 * A sweet smokey flavor throughout the pork!


## Command Usage Reference

```
Usage: pulledpork.pl [-dEgklnRTPVvv? -help] -c <config filename> -o <rule output path>
 -O <oinkcode> -s <so_rule output directory> -D <Distro> -S <SnortVer>
 -p <path to your snort binary> -C <path to your snort.conf> -t <sostub output path>
 -h <changelog path> -H <signal_name> -I (security|connectivity|balanced) -i <path to disablesid.conf>
 -b <path to dropsid.conf> -e <path to enablesid.conf> -M <path to modifysid.conf>
 -r <path to docs folder> -K <directory for separate rules files>

 Options:
 -help/? Print this help info.
 -b Where the dropsid config file lives.
 -C Path to your snort.conf
 -c Where the pulledpork config file lives.
 -d Do not verify signature of rules tarball, i.e. downloading fron non VRT or ET locations.
 -D What Distro are you running on, for the so_rules
    For latest supported options see http://www.snort.org/snort-rules/shared-object-rules
    Valid Distro Types:
	Alpine-3-10
	Centos-6, Centos-7, Centos-8
	Debian-8, Debian-9, Debian-10
	FC-27, FC-30
	FreeBSD-11, FreeBSD-12
	OpenBSD-6-2, OpenBSD-6-4, OpenBSD-6-5 
	OpenSUSE-15-0, OpenSUS-15-1, OpenSUSE-42-3
	RHEL-6, RHEL-7, RHEL-8
	Slackware-14-2
	Ubuntu-14-4, Ubuntu-16-4, Ubuntu-17-10, Ubuntu-18-4
 -e Where the enablesid config file lives.
 -E Write ONLY the enabled rules to the output files.
 -g grabonly (download tarball rule file(s) and do NOT process)
 -h path to the sid_changelog if you want to keep one?
 -H Send signal_name to the pids listed in the config file (SIGHUP or SIGUSR2)
 -I Specify a base ruleset( -I security,connectivity,or balanced, see README.RULESET)
 -i Where the disablesid config file lives.
 -k Keep the rules in separate files (using same file names as found when reading)
 -K Where (what directory) do you want me to put the separate rules files?
 -l Log Important Info to Syslog (Errors, Successful run etc, all items logged as WARN or higher)
 -L Where do you want me to read your local.rules for inclusion in sid-msg.map
 -m where do you want me to put the sid-msg.map file?
 -M where the modifysid config file lives.
 -n Do everything other than download of new files (disablesid, etc)
 -o Where do you want me to put generic rules file?
 -O Define the oinkcode on the command line (necessary for some users)
 -p Path to your Snort binary
 -P Process rules even if no new rules were downloaded
 -R When processing enablesid, return the rules to their ORIGINAL state
 -r Where do you want me to put the reference docs (xxxx.txt)
 -S What version of snort are you using (2.8.6 or 2.9.0) are valid values
 -s Where do you want me to put the so_rules?
 -T Process text based rules files only, i.e. DO NOT process so_rules
 -u Where do you want me to pull the rules tarball from
    ** E.g., ET, Snort.org. See pulledpork config rule_url option for value ideas
 -V Print Version and exit
 -v Verbose mode, you know.. for troubleshooting and such nonsense.
 -vv EXTRA Verbose mode, you know.. for in-depth troubleshooting and other such nonsense.
 -w Skip the SSL verification (if there are issues pulling down rule files)
 -W Where you want to work around the issue where some implementations of LWP do not work with pulledpork's proxy configuration.
 ```


## Basic Usage Examples

A simple example of how to use PulledPork would be to specify all of your configuration directives inside of the
`PulledPork.conf` file.  Specifically for minimal function, i.e. NO Shared Object rule processing you must define 
at a minimum the `rule_file`, `oinkcode`, `temp_path`, `tar_path`, and `rule_path` values.  Below are some examples of this.

```bash
./pulledpork.pl -o /usr/local/etc/snort/rules/ -O 12345667778523452344234234 \
  -u http://www.snort.org/reg-rules/snortrules-snapshot-2973.tar.gz \
  -i disablesid.conf -T -H
```

The above will fetch the `snortrules-snapshot-2973.tar.gz` tarball from snort.org using the specified `oinkcode` of 
`12345667778523452344234234` and put the rules files from that tarball into the output path of 
`/usr/local/etc/snort/rules/` while the `-i` option tells pulledpork where the
`disablesid.conf` lives, and the `-T` option tells pulledpork to not process for any shared object rules and the final
`-H` option tells pulledpork to send a `Hangup` signal to the snort pid that you defined in the `pulledpork.conf`.

```bash
./pulledpork.pl -c pulledpork.conf -i disablesid.conf -T -H
```

Similar to the first example but all options specified in the `pulledpork.conf` file (other than `disablesid` and `-H`)...

```bash
./pulledpork.pl -c pulledpork.conf -i disablesid.conf \
  -m /usr/local/etc/snort/sid-msg.map -Hn
```

The above will simply read the disablesid and disable as defined, then send a `Hangup` signal after generating the `sid-msg.map`
at the specified location without downloading anything.
Highly useful when tuning / making changes etc..

Next example, snort inline with rules that we want to drop and disable, then `HUP` our daemons after creating a `sid-msg.map`
and writing change info to `sid_changes.log`!

```bash
./pulledpork.pl -c pulledpork.conf -i disablesid.conf -b dropsid.conf \
  -m /usr/local/etc/snort/sid-msg.map -h /var/log/sid_changes.log -H
```

Next example, same as the previous but specifying that we want to run the default "security" based ruleset
and that we want to enable rules specified in `enablesid.conf`.

```bash
./pulledpork.pl -c pulledpork.conf -i disablesid.conf -b dropsid.conf \
  -e enablesid.conf -m /usr/local/etc/snort/sid-msg.map \
  -h /var/log/sid_changes.log -I security -H
```

Next example, same as the previous but specifying that we want to `-K` (Keep) the originationg tarball names.
and write them to `/usr/local/etc/snort/rules/`

```bash
./pulledpork.pl -c pulledpork.conf -i disablesid.conf -b dropsid.conf \
  -e enablesid.conf -m /usr/local/etc/snort/sid-msg.map \
  -h /var/log/sid_changes.log -I security -H -K /usr/local/etc/snort/rules/
```

For users of Suricata, the same steps are necessary for where your installation files reside, but all that pulledpork needs to process
rule files is the `-S` flag being set to `suricata-3.1.3` or whatever version of suricata you are using

```bash
./pulledpork.pl -c pulledpork.conf -S suricata-3.1.3
```

Pulledpork "should" work with Suricata and ET/ETPro rules. However there is no support for Talos rules to run on Suricata.

## Special Notes Section

Please note that pulledpork runs rule modification (enable, drop, disable, modify) in that order by default..

1. enable
2. drop
3. disable

This means that disable rules will always take precedence.. thusly if you specify the same `gid:sid` 
in enable and disable configuration files, then that sid will be disabled.. keep this in mind 
for ranges also!  However, you can specify a different order using the state_order keyword in the
master config file.

I'll probably add more info later, the `--help` or `--?` will display all runtime options and the `pulledpork.conf` is
pretty well annotated... so if you can't figure it out... try harder!  And once you figure it out, please feel 
free to contribute with additional readme / help foo.. thx!

As a side note, I would like to give a shout-out to my buddy Bruce for aiding in the naming of PulledPork!
"hopefully that will shut him up ;-)"

J
