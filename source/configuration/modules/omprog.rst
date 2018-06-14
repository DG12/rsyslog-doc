omprog: Program integration Output module
=========================================

**Module Name:    omprog**

**Available since:   ** 4.3.0

**Author:**\ Rainer Gerhards <rgerhards@adiscon.com>

**Description**:

This module permits to integrate arbitrary external programs into
rsyslog's logging. It is similar to the "execute program (^)" action,
but offers better security and much higher performance. While "execute
program (^)" can be a useful tool for executing programs if rare events
occur, omprog can be used to provide massive amounts of log data to a
program.

Executes the configured program and feeds log messages to that binary
via stdin. The binary is free to do whatever it wants with the supplied
data. If the program terminates, it is re-started. If rsyslog
terminates, the program's stdin will see EOF. The program must than
terminate. The message format passed to the program can, as usual, be
modified by defining rsyslog templates.

Note that each time an omprog action is defined, the corresponding
programm is invoked. A single instance is **not** being re-used. There
are arguments pro and con re-using existing binaries. For the time
being, it simply is not done. In the future, we may add an option for
such pooling, provided that some demand for that is voiced. You can also
mimic the same effect by defining multiple rulesets and including them
(at the price of some slight performance loss).

 

**Module Parameters**:

-  **Template**\ [templateName]
    sets a new default template for file actions.

 

**Action Parameters**:

-  **binary**
   Mostly equivalent to the "binary" action parameter, but must contain
   the binary name only. In legacy config, it is **not possible** to
   specify command line parameters.

**Caveats/Known Bugs:**

-  None.

**Sample:**

The following command writes all syslog messages into a file.

Module (load="omprog") \*.\* action(type="omprog"
binary="/pathto/omprog.py --parm1=\\"value 1\\" --parm2=value2"
template="RSYSLOG\_TraditionalFileFormat")

**Legacy Configuration Directives**:

-  **$ActionOMProgBinary** <binary>
   The binary program to be executed.

**Caveats/Known Bugs:**

Currently none known.

This documentation is part of the `rsyslog <http://www.rsyslog.com/>`_
project.
Copyright © 2008-2014 by `Rainer
Gerhards <https://rainer.gerhards.net/>`_ and
`Adiscon <http://www.adiscon.com/>`_. Released under the GNU GPL version
3 or higher.
