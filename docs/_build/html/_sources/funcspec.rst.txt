Functional Overview
###################


Operation
*********

Modbus Device Communication
===========================

A Modbus **device** (e.g. industrial automation controller, PLC, RTU, meter, etc.) is connected to a serial **port** on the satellite terminal.

Relevant Modbus register values are mapped to **parameters** that can be used to generate periodic **reports** and real-time **alerts**.

Each **report** can include multiple data types and values, optimized into a small satellite message.  This delivers a much lower data cost compared to polling native Modbus registers over-the-air.

Each **alert** is set up to trigger on a **parameter** threshold value, either high/low or a change from the previously reported value.  Alerts can include additional parameter data in addition to the trigger parameter value.

**Parameters** can be queried remotely in addition to periodic reports and alerts, either individually or in groups.  This method is more efficient than polling native Modbus registers which require separate polls for discontiguous register ranges and/or different data types.

Configuration of **ports**, **devices**, **parameters**, **reports** and **alerts** is normally done by creating a *configuration file* called ``config.dat`` that is included in a build package to load on the satellite terminal prior to deployment in the field.

An **OTA API** provides satellite **message** constructs for remote configuration, query of parameters, and direct read/write of Modbus registers.  
These **message** structures are sent using Inmarsat's `IDP Messaging API <https://developer.inmarsat.com/content/isatdatapro-messaging-api>`_.

Periodic Reports
----------------

By default, each **report** is sent immediately when configured or after a terminal reset.  Subsequent reports are sent at the defined interval in *seconds* relative to the previous report.

Time of Day Reports
-------------------

A report can be *optionally* configured to send data from a specific time of day.  
Because having many devices send data at the exact same time causes problems on the satellite network, when using Time of Day reports you must configure a **window** to distribute the time of day measurements on the satellite network.  
In this case, the measurement is taken as a *snapshot* at the specific time of day, and then sent a short while afterward.

Time of Day reports from a given terminal will be sent at the same offset time within the window, each day.  However 2 terminals configured with same time of day and window, may transmit at different times relative to the other.

Error Handling
--------------

Modbus and serial communications errors are captured and sent as :ref:`alerts <error>`, as well as logged for diagnostic retrieval.  The following error scenarios are handled:

*	Successive read timeouts are trapped as *loss of communication* based on the configured Timeouts (Rx, Tx) after the configured number of retries. 
	Communication is assumed to be re-established after the next successful read.
*	Modbus protocol errors

.. _diagnostics-messages:

.. csv-table:: Diagnostics Messages
	:file: csvtables/diagnostics.csv
	:header-rows: 1
	:widths: 5, 20



Geolocation and Movement Alerts
===============================

Geolocation is an *optional* feature that uses a reference location and periodically checks the terminal's location using internal GNSS to determine if the terminal has moved more than a certain distance in a certain time.
The **reference location** is determined by either:

* the current location when geolocation alerts are configured
* remote configuration using the **OTA API**

To stop geolocation alerts from repeatedly sending once the terminal has moved, you must send an **acknowledgement of movement** to the terminal using the **OTA API**.

.. note::
	GNSS has inherent error sources, so the geolocation feature takes multiple samples that must exceed the distance for a configured amount of time before a location change alert is sent.


Low Power Operation
===================

Power saving is an *optional* feature that changes the behaviour of the satellite terminal to conserve power, trading off against certain responsiveness to changes.

The following table summarizes the differences between normal and low power mode:

.. _lpm-behaviour:

.. csv-table:: Low Power Mode Behaviour
	:file: csvtables/lpm-behaviour.csv
	:header-rows: 1


Metadata
========

The following *metadata* is optionally available for each **report** and **alert**, which can use the default settings or override on a per report/alert basis:

* ``latitude`` is the latitude component of geolocation, expressed in 1/1000th minutes.  **Divide by 60,000** to get *decimal degrees* up to 5 decimal places precision.
* ``longitude`` is the longitude component of geolocation, expressed in 1/1000th minutes.  **Divide by 60,000** to get *decimal degrees* up to 5 decimal places precision.
* ``speed`` is the speed component of geolocation, expressed in *knots*.
* ``altitude`` is the altitude component of geolocation, expressed in *meters*.
* ``distance`` is the distance traveled, in meters, in between 2 geolocation reports.
* ``timestamp`` is the time that the **report** or **alert** was triggered, in seconds since *1970-01-01T00:00:00Z*.
* ``paramTimestamp`` is the time that an individual **parameter** was read from the Modbus device, in seconds since *1970-01-01T00:00:00Z*.


Feature Summary
***************

The Modbus proxy service has been designed based on the following requirements:

General
=======

#.	The following hardware variants are supported:
	
	* ST6100 (satellite-only)
	* IDP-680/690 (satellite-only)
	* IDP-782 (cellular/satellite)
	* IDP-800 (self-powered satellite-only)

#.	The Modbus *user service* is completely self-contained in terms of documentation and examples for an integrator, without dependencies on non-core **LSF** services.  
	A typical user does not need to be intimately familiar with the ORBCOMM product documentation and Lua Service Framework.
	

Physical Communication Layer
============================

#.	Configurable to work on any of the supported terminal’s RS232 or RS485 interface(s).
#.	Supports serial baud rates on industry-standard settings from 1200 to 115200, as well as it supports configuration of parity, data bits and stop bits.
#.	If communication with a slave device is lost, the service sends a corresponding alert.


Data Communication (Modbus Protocol) Layer
==========================================

#.	Supports operation on one or multiple terminal serial ports (one Modbus network per serial port).
#.	Configurable for Modbus RTU or ASCII protocol variants (one variant per Modbus network).
#.	Configurable Modbus slave response timeout, with a default value set to 3 seconds.
#.	Supports multiple Modbus device (slave) addresses when using RS485 port on a multi-drop Modbus network.
#.	Configurable for 0-based (default, native) or 1-based (PLC) Modbus addressing per each slave device.
#.	The following types of Modbus addresses supported:
	
	* Coils
	* Discrete inputs
	* Analog inputs
	* Holding (aka analog input/output) register

#.	Handles Modbus protocol errors by logging to the log core service. Logs can be retrieved remotely utilizing corresponding log service messages.


Data Interpretation
===================

#.	Maps Modbus register values to parameters using the following configuration options per parameter:
	
	* ``encoding`` data type (signed/unsigned integer or 8/16/32 bits, Boolean, 32-bit floating point, ASCII)
	* ``length`` data size (number of contiguous registers)

#.	Configurable byte order and word order (Endianness) on a per device basis).
#.	Data sampling (Modbus polling) interval is configurable value on a per-device basis.
#.	If specified in the configuration for the specific parameter, the service generates alerts if a measurement value exceeds some threshold either high, low, or a change of X amount.


Data Reporting and Register Read/Write
======================================

#.	OTA reports are configurable to contain multiple (up to 100) different register data values of same or different type, as individual parameter data fields within a single message.
#.	Reports can be sent at a configurable interval.
#.	Time-of-day based register reads can be captured as a “snapshot” into a timestamped report with data delivery distributed over a small window to load balance the satellite network.
#.	Supports native/transparent Modbus read/write commands and responses OTA, on-demand, using a messaging API.
#.	Effectively allows “remapping registers”, since parameter data fields do not need to use contiguous register values.
#.	Timestamps can optionally be included in reports or alerts on a per-parameter basis. Timestamp (UNIX epoch format) shall be that of the most recent Modbus slave response prior to message transmission.
#.	Location can optionally be included in reports and alerts.  Location resolution is approximately +/- 1m (1/1000 minutes, recognizing that GPS accuracy is typically 5-10m), +/- 1 km/h, +/- 1-degree heading, and +/- 1m altitude


Alerts and Data Analytics
=========================

#.	Supports configuration of up to 245 unique alerts, triggered by interpreting specified parameter data values including:
	
	* High value threshold
	* Low value threshold
	* Return-to-normal sends an alert when the high or low value last reported exceeded threshold but returned to normal range
	* Change value relative to last read value


Geolocation
===========

#.	Optional location change alert based on configurable distance change relative to last reported location (persisting in time to prevent random GNSS errors triggering false alerts).


Low Power Operation
===================

#. The service supports optional power saving to support battery-powered installations.


Service Configuration
=====================

#.	The service contains no default polling, reporting or alert configurations.  It must be explicitly configured.
#.	The primary method of configuration is pre-configuration using a text file, loaded on the satellite terminal using its main console RS232 port.
#.	Supports manual local configuration using console commands on the main RS232 interface.  Console commands present a similar structure as the configuration text file.
#.	Supports over-the-air (OTA) configuration query and changes using a messaging API.
#.	Supports embedded software local configuration using the Lua Service Framework API.
#.	All configurations are stored in non-volatile memory, and persist satellite terminal reset.
#.	Optional configuration of power-saving mode to minimize energy consumption.


Diagnostics
===========

#.	Collects Modbus diagnostics data such as timeouts and errors, made available by polling debug and error logs remotely using the messaging API.


Lua Service Framework API
=========================

#.	The service provides an effective interface for integration with other services and Agents via properties, events, function calls and messages


