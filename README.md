IPv6 Plugin for Snort
=====================

This Snort Plugin contains a preprocessor to monitor IPv6 neighbor discovery
messages and adds new rule options for IPv6 specific signatures.

For more information see

 * http://ipv6-ids.de/
 * http://mschuette.name/wp/snortipv6/
 * https://redmine.cs.uni-potsdam.de/projects/snort-ipv6-plugin/wiki/Installation


Compilation
===========

The Plugin builds with automake and autoconf, but the setup is quite primitive.

It also requires the Snort sources for some header files
(obtain them e.g. with `apt-get source snort`) and it currently
uses `pkg-config` to retrieve Snort config options, i.e. it needs
an installed Snort package.

I use these commands to fetch the Snort source code and compile this plugin (on Debian):

    (cd /tmp; apt-get source snort)
    automake
    autoconf
    ./configure --prefix=/usr CFLAGS="-I/tmp/snort-2.9.2.2/src/dynamic-preprocessors/include"
    make
    make install

To test the correct compilation activate the preprocessor (see below) and check if
it is loaded.

If the configuration flags do not match the configuration flags of your Snort
installation then the plugin loading should abort with an error like this:

    ERROR size 856 != 792
    ERROR: Failed to initialize dynamic preprocessor: IPv6 Preprocessor version 1.3.0 (-2)


Installation
============

Enable Preprocessor
-------------------

There are two steps to use a plugin:
1. Snort has to load the shared library. Either copy the library into the directory
configured with `dynamicpreprocessor directory /some/path`, or selectively load the library
file with `dynamicpreprocessor file /some/path/lib_ipv6_preproc.so`.
2. The preprocessor has to be enabled by adding the configuration line
`preprocessor ipv6`

Now the IPv6 preprocessor should be listed among the other loaded preprocessors
when starting Snort, and it should print its own summary when exiting Snort.

Add Preprocessor Rules
----------------------

Newer Snort versions use decoder and preprocessor rules by default.
So in order to use the plugin one has to add its preprocessor rules to
Snort's configuration files.
`etc/preprocessor.rules` and `etc/gen-msg.map` contain basic metadata on
the plugin's events and should be appended to the system's preprocessor.rules
and gen-msg.map files.

Configure Preprocessor
----------------------

The `preprocessor ipv6` line in snort.conf takes several optional parameters
to provide further information about the network.

Two useful parameters are the `router_mac` and the `net_prefix`.
If these are set then the preprocessor can perform additional checks
and raise an alert if it sees a rogue router or wrong network prefixes on-link.

Example:

    preprocessor ipv6: \
      router_mac 00:00:0C:01:02:03 00:00:0C:01:02:04 \
      net_prefix 2001:db8:1:2::/64

Add Signatures
--------------

The file `etc/ipv6.rules` contain some experimental rules for IPv6 network operation.
These can be tested by including the file in snort.conf, but they should be read
first because they need some customization (e.g. the DHCPv6 rules should be
disabled in a DHCPv6-managed network).
