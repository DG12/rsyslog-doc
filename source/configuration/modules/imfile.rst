imfile: Text File Input Module
==============================

.. index:: ! imfile 

===========================  ===========================================================================
**Module Name:**             **imfile**
**Author:**                  `Rainer Gerhards <https://rainer.gerhards.net/>`_ <rgerhards@adiscon.com>
===========================  ===========================================================================

This modul provides the ability to convert any standard text file
into a syslog
message. A standard text file is a file consisting of printable
characters with lines being delimited by LF.

The file is read line-by-line and any line read is passed to rsyslog's
rule engine. The rule engine applies filter conditions and selects which
actions needs to be carried out. Empty lines are **not** processed, as
they would result in empty syslog records. They are simply ignored.

As new lines are written they are taken from the file and processed.
Please note that this happens based on a polling interval and not
immediately. The file monitor support file rotation. To fully work,
rsyslogd must run while the file is rotated. Then, any remaining lines
from the old file are read and processed and when done with that, the
new file is being processed from the beginning. If rsyslogd is stopped
during rotation, the new file is read, but any not-yet-reported lines
from the previous file can no longer be obtained.

When rsyslogd is stopped while monitoring a text file, it records the
last processed location and continues to work from there upon restart.
So no data is lost during a restart (except, as noted above, if the file
is rotated just in this very moment).

Module Parameters
-----------------

.. index:: 
   single: imfile; PollingInterval
.. function:: PollingInterval seconds

   This is a global setting. It specifies how often files are to be
   polled for new data. The time specified is in seconds. The default
   value is 10 seconds. Please note that future releases of imfile may
   support per-file polling intervals, but currently this is not the
   case. If multiple PollingInterval statements are present in
   rsyslog.conf, only the last one is used.
   
   A short poll interval provides more rapid message forwarding, but
   requires more system resources. While it is possible, we stongly
   recommend not to set the polling interval to 0 seconds. That will
   make rsyslogd become a CPU hog, taking up considerable resources. It
   is supported, however, for the few very unusual situations where this
   level may be needed. Even if you need quick response, 1 seconds
   should be well enough. Please note that imfile keeps reading files as
   long as there is any data in them. So a "polling sleep" will only
   happen when nothing is left to be processed.

Action Parameters
-----------------

.. index:: 
   single: imfile; File
.. function:: File </path/to/file>

   **(Required Parameter)**
   The file being monitored. So far, this must be an absolute name (no
   macros or templates)

.. index:: 
   single: imfile; Tag
.. function:: Tag <tag:>

   **(Required Parameter)**
   The tag to be used for messages that originate from this file. If
   you would like to see the colon after the tag, you need to specify it
   here (as shown above).

.. index:: 
   single: imfile; StateFile
.. function:: StateFile <name-of-state-file>

   Rsyslog must keep track of which parts of the to be monitored file
   it already processed. This is done in the state file. This file
   always is created in the rsyslog working directory (configurable via
   $WorkDirectory). Be careful to use unique names for different files
   being monitored. If there are duplicates, all sorts of "interesting"
   things may happen. Rsyslog currently does not check if a name is
   specified multiple times. Note that when $WorkDirectory is not set or
   set to a non-writable location, the state file will not be generated.

.. index:: 
   single: imfile; Facility
.. function:: Facility <facility>

   The syslog facility to be assigned to lines read. Can be specified
   in textual form (e.g. "local0", "local1", ...) or as numbers (e.g.
   128 for "local0"). Textual form is suggested. Default  is "local0".

.. index:: 
   single: imfile; Severity
.. function:: Severity

   The syslog severity to be assigned to lines read. Can be specified
   in textual form (e.g. "info", "warning", ...) or as numbers (e.g. 4
   for "info"). Textual form is suggested. Default is "notice".

.. index:: 
   single: imfile; PersistStateInterval
.. function:: PersistStateInterval [lines]

   Specifies how often the state file shall be written when processing
   the input file. The **default** value is 0, which means a new state
   file is only written when the monitored files is being closed (end of
   rsyslogd execution). Any other value n means that the state file is
   written every time n file lines have been processed. This setting can
   be used to guard against message duplication due to fatal errors
   (like power fail). Note that this setting affects imfile performance,
   especially when set to a low value. Frequently writing the state file
   is very time consuming.

.. index:: 
   single: imfile; ReadMode
.. function:: ReadMode [mode]

   This mode should defined when having multiline messages. The value
   can range from 0-2 and determines the multiline detection method.
   0 (**default**) - line based (Each line is a new message)
   1 - paragraph (There is a blank line between log messages)
   2 - indented (New log messages start at the beginning of a line. If a
   line starts with a space it is part of the log message before it)

.. index:: 
   single: imfile; MaxLinesAtOnce
.. function:: MaxLinesAtOnce [number]

   This is useful if multiple files need to be monitored. If set to 0,
   each file will be fully processed and then processing switches to the
   next file (this was the default in previous versions). If it is set,
   a maximum of [number] lines is processed in sequence for each file,
   and then the file is switched. This provides a kind of mutiplexing
   the load of multiple files and probably leads to a more natural
   distribution of events when multiple busy files are monitored. The
   **default** is 1024.

.. index:: 
   single: imfile; MaxSubmitAtOnce
.. function:: MaxSubmitAtOnce [number]

   This is an expert option. It can be used to set the maximum input
   batch size that imfile can generate. The **default** is 1024, which
   is suitable for a wide range of applications. Be sure to understand
   rsyslog message batch processing before you modify this option. If
   you do not know what this doc here talks about, this is a good
   indication that you should NOT modify the default.

.. index:: 
   single: imfile;  Ruleset
.. function:: Ruleset <ruleset> 

   Binds the listener to a specific :doc:`ruleset <../../concepts/multi_ruleset>`.

Caveats/Known Bugs
------------------

* Only 100 files can be monitored. If more are needed, the source needs to be patched. See define MAX\_INPUT\_FILES in imfile.c

* The file must have a fixed name and location (directory). It is planned to add support for dynamically generating file names in the future.

* Powertop users may want to notice that imfile utilizes polling. Thus, it is no good citizen when it comes to conserving system power consumption. We are currently evaluating to move to inotify(). However, there are a number of subtle issues, which needs to be worked out first. We will make the change as soon as we can. If you can afford it, we recommend using a long polling interval in the mean time.

Configuration Example
---------------------

The following sample monitors two files. If you need just one, remove
the second one. If you need more, add them according to the sample ;).
This code must be placed in /etc/rsyslog.conf (or wherever your distro
puts rsyslog's config files). Note that only commands actually needed
need to be specified. The second file uses less commands and uses
defaults instead.

::

  module(load="imfile" PollingInterval="10") #needs to be done just once 

  # File 1 
  input(type="imfile" 
        File="/path/to/file1" 
        Tag="tag1"
        StateFile="statefile1" 
        Severity="error" 
        Facility="local7") 

  # File 2
  input(type="imfile" 
        File="/path/to/file2" 
        Tag="tag2"
        StateFile="statefile2") 

  # ... and so on ... #

Legacy Configuration Directives
-------------------------------

   The file being monitored. So far, this must be an absolute name (no
   macros or templates)

.. index:: 
   single: imfile; $InputFileName
.. function:: $InputFileName /path/to/file

.. index:: 
   single: imfile; $InputFileTag
.. function:: $InputFileTag tag:

   The tag to be used for messages that originate from this file. If
   you would like to see the colon after the tag, you need to specify it
   here (as shown above).

.. index:: 
   single: imfile; $InputFileStateFile
.. function:: $InputFileStateFile /path/to/state/file

   Rsyslog must keep track of which parts of the to be monitored file
   it already processed. This is done in the state file. This file
   always is created in the rsyslog working directory (configurable via
   $WorkDirectory). Be careful to use unique names for different files
   being monitored. If there are duplicates, all sorts of "interesting"
   things may happen. Rsyslog currently does not check if a name is
   specified multiple times.

.. index:: 
   single: imfile; $InputFileFacility
.. function:: $InputFileFacility facility

   The syslog facility to be assigned to lines read. Can be specified
   in textual form (e.g. "local0", "local1", ...) or as numbers (e.g.
   128 for "local0"). Textual form is suggested. Default  is "local0".

.. index:: 
   single: imfile; $InputFileSeverity
.. function:: $InputFileSeverity severity

   The syslog severity to be assigned to lines read. Can be specified
   in textual form (e.g. "info", "warning", ...) or as numbers (e.g. 4
   for "info"). Textual form is suggested. Default is "notice".

.. index:: 
   single: imfile; $InputRunFileMonitor
.. function:: $InputRunFileMonitor

   This activates the current monitor. It has no parameters. If you
   forget this directive, no file monitoring will take place.

   Multiple files may be monitored by specifying $InputRunFileMonitor
   multiple times.


.. index:: 
   single: imfile; $InputFilePollInterval
.. function:: $InputFilePollInterval seconds

   This is a global setting. It specifies how often files are to be
   polled for new data. The time specified is in seconds. The default
   value is 10 seconds. Please note that future releases of imfile may
   support per-file polling intervals, but currently this is not the
   case. If multiple $InputFilePollInterval statements are present in
   rsyslog.conf, only the last one is used.

   A short poll interval provides more rapid message forwarding, but
   requires more system ressources. While it is possible, we stongly
   recommend not to set the polling interval to 0 seconds. That will
   make rsyslogd become a CPU hog, taking up considerable ressources. It
   is supported, however, for the few very unusual situations where this
   level may be needed. Even if you need quick response, 1 seconds
   should be well enough. Please note that imfile keeps reading files as
   long as there is any data in them. So a "polling sleep" will only
   happen when nothing is left to be processed.

.. index:: 
   single: imfile; $InputFilePersistStateInterval
.. function:: $InputFilePersistStateInterval lines

   Specifies how often the state file shall be written when processing
   the input file. The default value is 0, which means a new state file
   is only written when the monitored files is being closed (end of

.. index:: 
   single: imfile; $InputFileReadMode
.. function:: $InputFileReadMode mode

.. index:: 
   single: imfile; $InputFileMaxLinesAtOnce
.. function:: $InputFileMaxLinesAtOnce number

   This is useful if multiple files need to be monitored. If set to 0,
   default is 10240.

.. index:: 
   single: imfile; $InputFileBindRuleset
.. function:: $InputFileBindRuleset ruleset

   Equivalent to: Ruleset
   Binds the listener to a specific
   :doc:`ruleset <../../concepts/multi_ruleset>`.

Caveats/Known Bugs
------------------

So far, only 100 files can be monitored. If more are needed, the source
needs to be patched. See define MAX\_INPUT\_FILES in imfile.c

Powertop users may want to notice that imfile utilizes polling. Thus, it
is no good citizen when it comes to conserving system power consumption.
We are currently evaluating to move to inotify(). However, there are a
number of subtle issues, which needs to be worked out first. We will
make the change as soon as we can. If you can afford it, we recommend
using a long polling interval in the mean time.

Legacy Example
--------------

The following sample monitors two files. If you need just one, remove
the second one. If you need more, add them according to the sample ;).
This code must be placed in /etc/rsyslog.conf (or wherever your distro
puts rsyslog's config files). Note that only commands actually needed
need to be specified. The second file uses less commands and uses
defaults instead.

::

  $ModLoad imfile # needs to be done just once 
  # File 1 
  $InputFileName /path/to/file1 
  $InputFileTag tag1: 
  $InputFileStateFile stat-file1

  $InputFileSeverity error 
  $InputFileFacility local7 
  $InputRunFileMonitor
  
  # File 2 
  $InputFileName /path/to/file2 
  $InputFileTag tag2:

  $InputFileStateFile stat-file2 
  $InputRunFileMonitor 
  # ... and so on ...
  # check for new lines every 10 seconds $InputFilePollingInterval 10
