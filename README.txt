Sendpage

What this tool does
-------------------
Sendpage is designed to speak SNPP on one end and TAP (or UCP) on the
other.  It gets pages from the network via SNPP, and then uses a modem
or a direct serial connection to deliver the pages to a Paging Central
(or "paging terminal").  Sendpage requires, for modem use, that you know
your PC's access number (which is not usually advertised by your paging
provider), and you need to know the PINs of the pagers you want to deliver
pages to.  All of this information is known by your paging provider.
If you ARE a paging provider, your job is much easier.  ;)

Quick Start (for RPM-based Linux distro)
----------------------------------------
Why are you reading this README?  Just run "rpmbuild -ta inkscape-*.tar.gz"
There is a .spec file included for easy building.  :)  I've only tested it
on SuSE 9.2, so please send me patches if you uncover problems with other
distros.

Quick Start (for non-RPM Linux distro)
--------------------------------------

   Requirements.

	- Net::SNPP (part of the libnet Perl module bundle)
	- Device::SerialPort version 1.02+  (or Win32::SerialPort, untested)
	- MailTools (provides Mail::Send)

   Installation.

	- run the following commands:

		perl Makefile.PL
		make
		make install

	- copy 'sendpage.cf' into /etc, and edit to your needs
	- copy 'email2page.conf' into /etc, and edit to your needs
	- copy 'snpp.conf' into /etc, and edit to your needs
	- if you use SysV-style init scripts, you can use "sendpage.init"
	  to start and stop sendpage.  Copy it to where you keep your rc
	  files.
	- create the user "sendpage"
	- figure out which group ID can write to your locks directory
	  (usually "uucp" can write in "/var/lock")
	- figure out which group ID can write to your modem device
	  (usually "tty" can write to "/dev/ttyS0")
	- make the queue area:

		mkdir -p /var/spool/sendpage
		chown sendpage /var/spool/sendpage
		chmod og-rwx /var/spool/sendpage

	- start the sendpage daemon as the root user (it will change
	  user id to 'sendpage'):

		sendpage -bd

	- go check your /var/log/messages for the syslog line saying
	  that sendpage started up:

		starting Queue Manager and SNPP listener (sendpage vX.X.X)

	- you can also use the "sendpage.init" file for SysV-style init
	  script control.	
	- make sure you have an entry in /etc/services for "snpp", which
	  should be port 444.

   Sending a page.

	- run the "snpp" command to send yourself a test page:

		snpp -d -f me@yourdomain.org -m 'hi there!' yourPIN@yourPC

	- if you want to write pages directly to the paging queue instead of
	  using SNPP, you can use "sendpage" itself, but you must be root:

		sendpage -f me@place.org -m 'hello again!' yourPIN@yourPC

	  the sendpage daemon must be running for this to actually be delivered.

	

Long Version
------------
Sendpage (the C program) was originally written in 1995 almost entirely
by Mark Fullmer.  Version "7" was when I became aware of it picked up
the code, since Mark didn't want to maintain it any more.  I developed
a "configure" script to make compiling easier, started working on merging
some features that other people had written, and stomping out a few bugs.
I released a series called "0.8", which still had a bunch of problems,
but seemed to mostly work for most people.  Then after not looking at the C
code for about a year, it was getting rather dusty.  I was about to
work on porting it up to RedHat 6.1 and realized it would be easier to
redesign it from scratch.  I decided to use Perl because of all the great
modules I could use to help me get my job done.  And then I could spread
the blame around when things didn't work right.  ;)

And at this point I'd like to apologize here for my sloppy Perl modules.
I was new to writing *modules* under Perl, so I fear that I've violated
module and CPAN specs left and right.  I will try to correct any of these
problems as I learn about them.  My final goal is to produce a set of
modules that would be useful to any other packages that need to send
pages directly, and could be installed via CPAN. 

To get it started you'll need the following Perl modules installed:

	- a serial port module.  Either
		Device::SerialPort (for POSIX (unix))
			-or-
		Win32::SerialPort (for Windoze)
	- Mail::MailTools
	- Net::SNPP

You can get these from http://cpan.perl.org/

Now, build sendpage itself.  Right now, just do:
	cd sendpage-X.X.X
	perl Makefile.PL
	make
	make install

to install this package.  The important man page (sendpage's) is done.
The others are mostly just overviews.  I need to finish those.  If you
want to put the files somewhere besides where Perl wants to put them,
you can always create the Makefile is "perl Makefile.PL PREFIX=/usr/local"
or where ever you want the files to be forced to go.

Make sure you create a sendpage user, and identify the group who has
write access to /var/lock, and the group who has write access to your
modem device.

Once that's working, take a look at "sendpage.cf", and configure it
for your modem, paging central, etc.  Turn on the debugging if you're
interested.  Note that "sendpage.cf" needs to live in /etc, or where ever
you run "sendpage -C" with.  Please note, that you can safely ignore any
"Attempt to free unreferenced scalar." messages you get while shutting
down sendpage if you have debugging turned on.  This is just Perl
complaining about the paging centrals for some reason.  Hopefully these
should be gone now.

Install and edit the "email2page.conf" file for the email-to-page
converting tool.  Documentation is in the conf file for that one right now.

Install and edit the "snpp.conf" file for the snpp's default
server.  Documentation is in the conf file for that one right now.

Now there is the "sendpage" daemon itself.  Run (as root) the command:
	sendpage -bd
to start up the daemon mode.  (Make sure that "sendpage" is in the PATH
for the root user.)  The SNPP server connections and Paging Centrals will
lose root privs when the fork, so that should be safe.  "sendpage -bs" will
stop sendpage.  The file "sendpage.init" has been included to run it as a
SysV-style init script, which takes "start" and "stop" arguments.  This
script assumes that sendpage is installed in /usr/bin, so be sure to
change it if you need to.

To put a page into the queue, run "snpp RECIP" where RECIP is either a 
valid alias "recip" section in /etc/sendpage.cf, or has the form PIN@PC,
where the PC is a valid "pc" section in /etc/sendpage.cf.  It will deliver
directly to the named PIN.  "snpp" will read from STDIN for the message,
unless you use the '-m' flag.  To use "snpp" from other networked machines,
you'll need to edit sendpage.cf to bind snpp to "0.0.0.0" or some other
network interface, and then use "snpp -s SERVER" from the remote client.
Be sure to read and set at least one "snpp-acl" line so that your remote
machine will have permission to connect to the SNPP server.

To force a queue run immediately, run (as root) "sendpage -q".  If you have
debugging turned on, it should be flooding your syslog with lots of fun
information while delivering the pages.

Look in the "docs" and "examples" directories for various misc information
files, including the TAP specifications, the SNPP RFC, and a list of
the known paging centrals in the file "PagingCentrals.txt".  See if the
PCs for your area are listed.  If not, and you locate them, you should
email them to me, and I'll update this list.  There is also a searchable
database of these available at the website below.  For an example
of how to set up sendpage as an email gateway, read "sendmail.txt".
For an example of how to make a web page to send pages, look at the
"sendpage.php" PHP script.  Also in this directory is the beginnings of
my official "Sendpage Manual" in LyX format.

Logging
-------

By default, sendpage uses syslog facility "daemon" to do it's reporting,
with all messages set to "info" or higher.  This should be seen by any
sanely configured syslog daemon.

For myself, I wanted to keep my sendpage syslogs out of my general syslog
file ("/var/log/messages"), and I wanted debugging to be reported at
the syslog level "debug".  I changed the facility to "local6", and the
minimum level to "debug" with the following lines in my sendpage.cf:

syslog-facility = local6
syslog-minlevel = debug

and changed my /etc/syslog.conf looks like this:

*.info;mail.none;news.none;authpriv.none;local6.none  /var/log/messages 

local6.*                                              /var/log/sendpage
local6.=debug                                         /var/log/sendpage.debug
local6.info                                           /var/log/sendpage.info 

This way "messages" doesn't get anything from "local6" (sendpage), and I  
can watch the sendpage output in three ways: either everything going to  
local6 ("/var/log/sendpage"), JUST the stuff *at* the "debug" level
("/var/log/sendpage.debug"), or everything "info" and higher
("/var/log/sendpage.info").


Compatibility
-------------
Linux:	  This tool (and Device::SerialPort) was written and tested under
	  Linux.  There really shouldn't be any compatibility issues under
	  Linux.

Solaris:  Some headers under Solaris don't behave very well, so I had
	  to define "__sparc" explicitly in my patch to Device::SerialPort.
	  Also, under Solaris, many of the "make test" tests for
	  Device::SerialPort seem to hang and/or fail.  Most of these
	  seem safe to ignore as far as using sendpage with Device::SerialPort,
	  as the test failures seem to be with timings and/or flushes.
	  Most of that is unreliable anyway under Linux, so sendpage has
	  already worked around these bugs.

	  From the following URL:
	  http://www.stokely.com/unix.serial.port.resources/tutorials.html

		DTR Delay Problems: By default, Suns have a three second
		delay in toggling dtr. If your Sun has a zs serial port you
		can set the variable default_dtrlow to control the number of
		seconds of the delay. If the variable is set to zero, dtr
		can be toggled many times a second. For example, in
		/etc/system add the line "set zs:default_dtrlow=1" to have
		a 1 second delay.  

		If your workstation has an se serial port, the /etc/system
		line should be "set se:se_default_dtrlow = 1". However, in
		initial versions of the se driver, the delay was the value
		of (se_default_dtrlow + 1) seconds. If you have this version
		of the se driver, don't set the value to -1 in /etc/system or
		the port will hang on open. If you need to toggle dtr quickly,
		you can still set the value to -1 after the terminal is
		opened by using adb to set the variable manually.

		All this is Sun bugid 4230310, fixed by patch 105924-09 or
		higher. The patch makes se_default_dtrlow behave like
		default_dtrlow (i.e. setting se_default_dtrlow to 0 will allow
		rapid toggling of dtr instead of once per second). 

	  So, since I have zs serial ports on my Sun, I added the line:

		set zs:default_dtrlow=0

	  so I could toggle the DTR rapidly.

	  Also, I've seen trouble using Syslog correctly under Solaris.
	  Once again, this appears to be Perl's fault.  If it breaks
	  ("Your vendor has not defined the Sys::Syslog macro _PATH_LOG")
	  then just edit KeesLog.pm and remove the calls to "setlogsock".

AIX:	  Tested with 4.3.2, and works fine, some modem lines don't work.

*BSD:     Tested.  Carrier detection works after Sendpage 0.9.13.

HPUX:	  Untested so far...

SCO7:     Install notes from Pat Gunn:

          Removed system perl first (MUST happen before installing skunkperl
          or perl will become messed up)

          Then installed skunkperl from skunkware
          (stage.caldera.com/uw7/Packages/)
        
          Used CPAN module to upgrade prerequisites (didn't allow it to
          rebuild Perl itself. I have GCC and other gnu tools installed)

          Installed Device::Serialport (after these mods, it might work through
          CPAN, but you're better off doing it by hand so you can troubleshoot
          problems more easily)

          Installed sendpage.

          Created sendpage user, created /var/lock, set its group to uucp and
          changed its mode to 775, created sendpage spool directory, configured
          sendpage.conf and snpp.conf. Note that syslog does not appear to work
          with sendpage's logging component, so it's probably better to log to
          stderr. Change snpp.conf accordingly. It might be possible to fix
          this with some tinkering, but I'm not interested enough to take the
          time.

          The modem device on my system (serial port A) is /dev/term/00m,
          although I made a symlink to there from /dev/modem, and put
          /dev/modem in the configfile.


Porting Device::SerialPort
--------------------------
If you're trying to get Device::SerialPort to work on an untested platform,
please take a look at "configure.ac" for the list of files being included.
Basically, SerialPort.pm needs the following constants from .h files to
operate correctly:

Must have these:
	TIOCM_RTS
	TIOCM_DTR
	TIOCMBIS
	TIOCMBIC
	TIOCMGET

Optional: I have only found these on OpenBSD (and are required for OpenBSD):
	TIOCSDTR
	TIOCCDTR

Optional: I have only found *both* of these under Linux:
	TIOCINQ
	TIOCOUTQ

Optional: I have only found this one under Linux:
	TIOCSERGETLSR

Once you get SerialPort.pm running happily (you can set $DEBUG=1
near the top), try modifying (from the sendpage package) the script
"modemtest" in the "examples" directory to use your serial port with
a modem attached.  This script will try to examine all the possible
settings for your vendor's serial port, and will try to talk to the
modem using Device::SerialPort.  If you can get this script talking to
the modem, sendpage should operate just fine.


Misc
----
The TODO file lists a large number of things I still want to have done.

For further information, check out the sendpage web site at:
	http://sendpage.org/

For CVS updates, check out the sourceforge project at:
	http://sourceforge.net/projects/sendpage



Thanks for using sendpage!


-Kees Cook
 kees@outflux.net

# $Id: README 183 2005-02-24 01:21:07Z nemies $
