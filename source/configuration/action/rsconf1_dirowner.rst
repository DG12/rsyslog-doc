`back <rsyslog_conf_global.html>`_

$DirOwner
---------

**Type:** global configuration directive

**Default:**

**Description:**

Set the file owner for directories newly created. Please note that this
setting does not affect the owner of directories already existing. The
parameter is a user name, for which the userid is obtained by rsyslogd
during startup processing. Interim changes to the user mapping are not
detected.

**Sample:**

``$DirOwner loguser``

[`rsyslog.conf overview <rsyslog_conf.html>`_\ ] [`manual
index <manual.html>`_\ ] [`rsyslog site <http://www.rsyslog.com/>`_\ ]

This documentation is part of the `rsyslog <http://www.rsyslog.com/>`_
project.
Copyright © 2007 by `Rainer Gerhards <https://rainer.gerhards.net/>`_
and `Adiscon <http://www.adiscon.com/>`_. Released under the GNU GPL
version 2 or higher.
