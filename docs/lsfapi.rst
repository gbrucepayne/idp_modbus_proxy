LSF API Integration
###################

.. note::
	This section assumes the user is familiar with the ORBCOMM shell interface (**T404**) and Terminal API (**T405**).

Properties
==========

See :ref:`Configuration <config-properties>`


Shell Commands
==============

The shell commands for the service are used as an individual configuration utility, as a testing/development alternative to ``config.dat``.


Modbus Communications Configuration
-----------------------------------

* ``modbus config set [<port>|<device>|<parameter>|<property>]`` with:
	
	* ``port [name] [<baudRate> <parity> <mode>]``
	* ``device [deviceId] [<networkId> <port> <pollInterval> <byteOrder> <wordOrder> <serialRxTimeout> <serialTxTimeout> <plcBaseAddress>]``
	* ``parameter [paramId] [<deviceId> <registerType> <address> <length> <encoding> <mult>]``
	* ``property [pin] [paramId]``

* ``modbus config get [<all>|<port>|<device>|<parameter>|<properties>]``
* ``modbus config del [<all>|<port>|<device>|<parameter>|<properties>]``

Example commands::
	
	shell> modbus config set port=rs485 baudRate=9600 parity=none mode=rtu
	shell> modbus config set deviceId=1 networkId=1 port=rs485 pollInterval=10
	shell> modbus config set paramId=1 deviceId=1 registerType=analog address=31 encoding=int16


Reporting Configuration
-----------------------

* ``modbus reporting set [reportingId] [interval] <timeOfDay> <timeOfDayWindow> [paramIds] <msgBitmap>``
* ``modbus reporting get [<all>|<reportingId>]``
* ``modbus reporting del [<all>|<reportingId>]``

Example command::
	
	shell> modbus reporting set reportingId=1 interval=3600 paramIds=1,2,3,4,5


Alerts Configuration
--------------------

* ``modbus analytics set [analyticsId] [paramId] [<min> <max> <change>] <paramIds> <msgBitmap>``
* ``modbus analytics get [<all>|<analyticsId>]``
* ``modbus analytics del [<all>|<analyticsId>]``

Example command::
	
	shell> modbus analytics set analyticsId=1 paramId=1 maxON=350 maxOFF=300 paramIds=1,2,3,4,5


Events
======

None.


Functions
=========

None.

