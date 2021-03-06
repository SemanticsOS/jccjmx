Abstract
========

SSH-tunnel friendly monitoring of embedded Java Runtimes using MBeans,
Java Management Extensions (JMX) and Remote Method Invocation (RMI) for
Python and PyLucenet.

Rationale
=========

jccjmx is a convenience helper for JCC and PyLucene to create a JMX RMI
connector at runtime. The usual way to a connector with vmargs
'-Dcom.sun.management.jmxremote' (or similar) works only on startup.
Also this opens two ports (one for the connector and one for the RMI
registry). The RMI registry port is assigned dynamically which makes
firewall rules or SSH/SSL tunnels impossible.

jccjmx allows you to start a RMI and JMX connector programmatically
without restarting your Python application. The platform MBean server
provides live monitoring of JRE's memory usage (heap, caches), JRE's CPU
usage, Java threads, attached Python threads and more. You can even
profile your app (CPU profiling or memory profiling by class). The JDK
is shipped with two GUI programs for monitoring: jconsole and jvisualvm.
heck out the Oracle dosc for more information about jvisualvm:
http://docs.oracle.com/javase/6/docs/technotes/guides/visualvm/index.html

jccjmx is based on Daniel Fuchs' examples from his blog at Sun (now
Oracle). I've modified his code and ported it from a premain agent to a
standalone class.

Usage
=====

The import order is crucial! You must import and init lucene and jccjmx
in the correct order. Otherwise your process will segfault::

  >>> import lucene
  >>> import jccjmx

Initialize the VM for both packages. The second initVM() just adds the
CLASSPATH of jccjmx::

  >>> lucene.initVM() # doctest: +ELLIPSIS
  <jcc.JCCEnv object at 0x...>
  >>> jccjmx.initVM() # doctest: +ELLIPSIS
  <jcc.JCCEnv object at 0x...>

Create an agent that listens on port 12345. You should create just one
instane of JccJmxAgent during the life time of your application::

  >>> agent = jccjmx.JccJmxAgent(12345)

By default the agent is bound to 127.0.0.1. You can specificy another
hostname or IP address with jccjmx.JccJmxAgent("hostname", portnumber).

A RMI is created immediately and bound to "\*:port" but no agent is
listening yet. You have to activate is explicitly. This allows you to
delay the agent::

  >>> agent.isActive()
  False
  >>> agent.start()
  >>> agent.isActive()
  True
  >>> agent.stop()
  >>> agent.isActive()
  False

In order to connect from a remote host you need to know the service URL::

  >>> agent.getServiceURL()
  u'service:jmx:rmi://127.0.0.1:12345/jndi/rmi://127.0.0.1:12345/jmxrmi'

From a remote host::

  $ ssh -L12345:127.0.0.1:12345 server
  $ jconsole service:jmx:rmi://127.0.0.1:12345/jndi/rmi://127.0.0.1:12345/jmxrmi

Security
========

The RMI registry is always bound to all possible network devices. This
shouldn't be an issue but I can't guarantee it.

jccjmx doesn't use SSL to authenticate clients and encrypt the
connection. It's up to you to use an encrypted tunnel. Future versions
of jccjmx may support SSL.

Trouble shooting
================

Connection refused
------------------

You may get a connection refused error when the RMI hostname isn't set
correctly. jccjmx sets the system property unless it is already set. You
can force a correct hostname with e.g.::

 -Djava.rmi.server.hostname=127.0.0.1

IPv4 vs. IPv6
-------------

Java prefers IPv6 connections over IPv4 connections and usually binds to
IPv6 TCP. If you are having trouble with a mixed network you can force
the JRE to prefer IPv4 with::

 -Djava.net.preferIPv4Stack=true

Sources
=======

https://blogs.oracle.com/jmxetc/entry/connecting\_through\_firewall\_using\_jmx
https://blogs.oracle.com/jmxetc/entry/more\_on\_premain\_and\_jmx
https://blogs.oracle.com/jmxetc/entry/jmx\_connecting\_through\_firewalls\_using

Authors
=======

Christian Heimes Daniel Fuchs (original author of the JMX agent)

Copyright
=========

Original java code::

Copyright 2007 Sun Microsystems, Inc. All Rights Reserved.

JCC wrapper, start/stop features::

 Copyright (C) 2012 semantics GmbH

 semantics Kommunikationsmanagement GmbH Viktoriaallee 45 D-52066 Aachen
 Germany

 Tel.: +49 241 89 49 89 29 eMail: info(at)semantics.de
 http://www.semantics.de/
