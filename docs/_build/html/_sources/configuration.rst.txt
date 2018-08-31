Configuration
#############

The Modbus service implements 3 primary functional modules: communication, reporting and analytics.  Configuration of all three modules is done using any of:

* ``config.dat`` file loaded locally on the console
* console shell commands
* **OTA API**
* **LSF API** 

Additionally, optional functional modules support geolocation movement alerts, power-saving operation, and diagnostics logs.

.. _configfile:

The Configuration File
**********************

The most practical way of configuring for deployment is to create a ``config.dat`` file and build it into a **.idppkg** package file.

**Comments** can be added to the ``config.dat`` file by enclosing in ``/*`` and ``*/``. 

A simple example ``config.dat`` file could look like :ref:`this <example-config-dat>`.


Shell Commands
**************

For initial testing of the service, you can use :ref:`shell commands <shell_commands>` on the main serial port of the terminal to configure individual **ports**, **devices**, **parameters**, **reports** and **alerts**.

The *shell* commands can be queried iteratively using the ``?`` operator::

	shell> modbus ?

An example *shell* command to configure a **parameter** would look like::

	shell> modbus config set parameter paramId=1 deviceId=1 registerType=analog address=31 encoding=int16


Communications Configuration
****************************

The Modbus service is typically configured to interface with a single industrial device, but can support connection to a Modbus network on each available serial port of the satellite terminal.  Modbus is a master-slave protocol, and the satellite terminal acts as the master and polls device slave(s) attached to the RS232 and/or RS485 serial port.

The communication hierarchy to be configured is:

* ``port`` defines the physical serial port used to connect to the Modbus device or network.
* ``device`` is a Modbus slave on a given Port, assigned a unique ID within the service context.  Multiple Devices can be associated with a single Port.
* ``parameter`` associates to an individual data value obtained from Modbus register(s) on a given Device.
* ``property`` *optionally* associates to a **parameter** to expose that Modbus data value individually to the **OTA API** and/or **LSF API**.

.. _ports:

Ports
=====

The satellite terminal supports one or more serial ports depending on the hardware model (see Terminal Reference Table). Each ``port`` can be configured to talk to a Modbus slave device or, in the case of RS485 multi-drop, a network of devices.

Each **port** has the following parameters:

	* ``port`` (*required*)
		
		``rs232`` or ``rs232aux`` or ``rs485`` depending on the physical port available on the terminal:

		.. table:: Ports available on satellite terminal models

			+--------------------+-----------+--------------+-----------+
			| **Terminal Model** | **rs232** | **rs232aux** | **rs485** |
			+--------------------+-----------+--------------+-----------+
			| **ST6100**         |     -     |              |     -     |
			+--------------------+-----------+--------------+-----------+
			| **IDP-680/690**    |     -     |              |     -     |
			+--------------------+-----------+--------------+-----------+
			| **IDP-782**        |     -     |       -      |     -     |
			+--------------------+-----------+--------------+-----------+
			| **IDP-800**        |     -     |              |           |
			+--------------------+-----------+--------------+-----------+

	* ``baudRate`` (*required*)
		
		A number value in the range ``1200``, ``2400``, ``4800``, ``9600``, ``19200``, ``38400``, ``57600``, ``115200``

	* ``parity`` (*required*)
		
		``even``, ``odd`` or ``none``

	* ``mode`` (*required*)
		
		The variant of Modbus protocol used: ``rtu`` or ``ascii``

		.. note::
			The selection of ``ascii`` or ``rtu`` mode does not affect data transmission costs.

.. _devices:

Devices
=======

Each Modbus device attached to a serial port is assigned a unique ID within the service.  If interfacing to multiple Modbus devices on a multi-drop RS485 network, each ``device`` will have a “network” ID unique on that multi-drop network.

Each **device** has the following configuration parameters:

	* ``deviceId`` (*required*)

		A number from **1..255**, globally unique identifier for the device within the service, used to identify a physical device on the attached Modbus network. 
		Note this may be different from the Modbus *Device ID* (*slave address*) configured on the physical device.

	* ``networkId`` (*required*)

		A number from **1..247** mapping to the Modbus slave address of the physical device on the Modbus network. 
		Typically, this will be specified by the device manufacturer and often uses a default value of 1.

	* ``port`` (*required*)

		Serial communication **port** on which the Modbus Slave device is connected ``rs232``, ``rs232aux`` or ``rs485``.

	* ``pollInterval`` (*required*)

		Interval in seconds from **0..604800** to poll the configured set of register data from a particular device. 

		* 0 = disabled
		* 604800 = weekly

		.. note::
			This value by itself does not affect satellite data use, and typically this is configured at a fast interval in order to capture value changes in real-time to supplement periodic **reports** with **alerts**.

	* ``byteOrder`` (*required*)

		``msb`` (*most significant bit*) or ``lsb`` (*least significant bit*) is a sequential order in which bytes are sent over serial link, and is generally defined by the Modbus device manufacturer (based on the microprocessor they use).

		.. note::
			Most Significant Bit (msb) format is also called big-endian or network byte order, and is the typical/default method used.

	* ``wordOrder`` (*required*)

		``msw`` (*most significant word*) or ``lsw`` (*least signficant word*) similar to the *byteOrder* principle, Modbus registers are stored as 16-bit *words* and the wordOrder is a sequential order in which words, formed by bytes, are to be interpreted to read or write the data value. 
		When polling a block of contiguous registers, the word order is important to be able to interpret the data, for example to convert to a standard integer number for transmission or analytics.

	* ``serialRxTimeout`` (*optional*)

		Time in milliseconds from **100..30000** to wait for read data from the Modbus device. 
		If the Modbus device fails to send a byte serialRxTimeout milliseconds after the last byte, a read timeout error will be raised.

	* ``serialTxTimeout`` (*optional*)

		Time in milliseconds from **100..30000** to wait for the acknowledgement of transmission to the Modbus device. 
		If the Modbus device fails to send an acknowledgement within serialTx Timeout milliseconds after the last byte has been sent, a write timeout error will be raised.

	* ``plcBaseAddress`` (*optional*)

		A flag value set to **1** if the Modbus device does not use zero-based addressing.
		Some Modbus devices (typically Programmable Logic Controllers **PLC**) define register addressing with an offset of 1 (e.g. register 1 on the PLC is actually data address 0).
		
		Setting plcBaseAddress equal to 1 allows the configuration of Parameters that correspond directly to the device manufacturer’s documentation.

	* ``retries`` (*optional*)

		The number of retries **1..10** that determines how many data transmission attempts will be made before raising an error. The default value is 1.

.. _parameters:

Parameters
==========

Each **parameter** is a uniquely identified data value that maps to a Modbus register value or contiguous register range of a common type used to define a single value.  
Certain data types support analytics that can provide filtering and escalation of critical changes to supplement conventional polled data collected by the centralized application.  

A **parameter** can also be mapped to a **property** :ref:`Properties <properties>`, which are exported for direct access by either the **OTA API** or **LSF API**.

Parameters have the following configurable attributes:

	* ``paramId`` (*required*)

		A number from **1..255** used to uniquely identify this particular device and parameter within the service for the data value of interest.

	* ``deviceId`` (*required*)

		A number from **1..255** mapping to the :ref:`Device <devices>` the parameter is associated with.

	* ``registerType`` (*required*)

		A register type ``coil``, ``input``, ``analog`` or ``holding``.

		Typically Modbus registers are defined based on the following ranges:

		.. table:: Modbus Register Ranges and Types
			:widths: 10, 10, 17, 5

			+---------------------+------------------+-----------------------------------+------------------+
			| **Register Number** | **Data Address** |       **Modbus Definition**       | **registerType** |
			+---------------------+------------------+-----------------------------------+------------------+
			| 1 .. 9999           | 0x0000 .. 0x270E | Discrete Output Coils             |    ``coil``      |
			+---------------------+------------------+-----------------------------------+------------------+
			| 10001 .. 19999      | 0x0000 .. 0x270E | Discrete Input Contacts           |    ``input``     |
			+---------------------+------------------+-----------------------------------+------------------+
			| 30001 .. 39999      | 0x0000 .. 0x270E | (Analog) Input Registers          |    ``analog``    |
			+---------------------+------------------+-----------------------------------+------------------+
			| 40001 .. 49999      | 0x0000 .. 0x270E | Holding Registers                 |    ``holding``   |
			+---------------------+------------------+-----------------------------------+------------------+

	* ``encoding`` (*required*)

		Defines how to interpret data from Modbus registers for analytics or reporting.
		
		* ``boolean`` converts 0 value to false and any non-zero value to true
		* ``int8`` converts register data to 8-bit signed integer value from -128..127
		* ``uint8`` converts register data to 8-bit unsigned integer value from 0..255
		* ``int16`` converts register data to 16-bit signed integer value from -32768..32767
		* ``uint16`` converts register data to 8-bit unsigned integer value from 0..65535
		* ``int32`` converts register data to 32-bit signed integer value from -2147483648..2147483647

		.. note::
			32-bit unsigned integer is not supported by some versions of the **LSF** and therefore is not supported by the Modbus service.

		* ``float32`` converts register data to a single precision floating number.

		.. note::
			Floating point numbers are sent as **string** data types in the **OTA API**

		* ``string`` converts register data into an ASCII format
		* ``base64string`` converts register data into a base64 string format
		* ``raw`` converts register data into a single byte array format

	* ``address`` (*required*)

		The *data address* or from **0..65535** for the starting register to read.

		.. note::
			If values are not reading properly from your Modbus device, you may need to set ``plcBaseAddress`` to 1.

	* ``length`` (*conditional*)

		The number of registers to read from **1..125** holding a single data value. 
		Since Modbus registers are 16-bits, length is implicitly=1 for encodingType up to 16 bits, and implicitly=2 for 32-bit encodingType.

		Length is *required* to be specified when encoding is configured as ``string``, ``base64string`` or ``raw``.

	* ``mult`` (*optional*)

		A multiplier from **1..100** applied when reading the encoded register value, typically for purposes of analytics, reporting or API access.  

		.. note::
			Conversion of floating point to a whole number is useful for analytics since floating point math is problematic in the **LSF**.

.. _properties:

Properties
==========

Properties are *optional* and intended to be used within the **LSF API** to directly read or write configuration settings and data parameters from the local console, over-the-air messaging interface or other LSF services, accessed by service-specific Property Identification Numbers (PIN).  The Modbus service allows you to assign an individual device data parameter to a PIN that is exported for access by the LSF.

Properties have the following configurable attributes:

	* ``paramId`` (*required*)

		The unique :ref:`Parameters <parameters>` id from **1..255** corresponding to the specific device and register value of interest.

	* ``pin`` (*required*)

		The unique Property Identifier Number from **10..255** where the parameter value can be read as a **property**.


Reporting Configuration
***********************

A **report** is used to send one or more **parameter** values on a periodic basis.  
The report interval can be stochastic based on an offset from when the terminal is powered up or reset, or based on time of day.

.. note::
	Time of day reports *must* be distributed over a population of terminals to minimize satellite packet loss and retransmissions that could affect network quality of service.  
	To accommodate fixed time reports, the defined Modbus registers are read at the configured time of day as a *snapshot*, but the report is distributed over a configurable window of time.  
	A given terminal will always report at the same time offset within the window, but any 2 terminals will have different offsets within the window.

Reports
=======

Reports are configured either locally via a ``config.dat`` file, manual console typed commands, **OTA API** or **LSF API**.

Reports have the following configurable attributes:

	* ``reportingId`` (*required*)

		A unique identifier from **1..255** corresponding to a defined set of **parameters**.

	* ``paramIds`` (*required*)
		A string of comma separated values of unique **parameter id(s)** that will be included in the report.

	* ``interval`` (*required*)

		The frequency of transmission in seconds from **0..604800**.  The timer starts as soon as the configuration is processed, or each terminal reboot.

		| 0 = disabled (do not send)
		| 604800 = weekly

	* ``timeOfDay`` (*optional*)

		The time of day in seconds from midnight UTC **0..86400** when a *snapshot* reading of the associated **parameters** will be done.
		
		| 0 = disabled (do not send)
		| 86400 – midnight (starting next day)

		.. note::
			The time of day report will be sent *after* the measurement, witin a window specified by ``timeOfDayWindow``.

	* ``timeOfDayWindow`` (*conditional*)

		A time period in seconds from **0..3600** that must be configured if using ``timeOfDay`` is non-zero.  
		This is used to delay reports that are scheduled to *snapshot* at a specific time of day, to avoid satellite network congestion.
		
		The *window* is established immediately after the scheduled timeOfDay during which a given terminal can report its snapshot parameters.

		.. note::
			A given terminal will transmit at the same offset each period.  But two different terminals with same ``timeOfDay`` may transmit at different times.

	* ``msgBitmap`` (*optional*)

		An ASCII hex string value from **"00".."FF"** used to override the default Bitmap and include/exclude optional message fields in this particular **report**.

		See: :ref:`msgBitmap <msgbitmap>`


Analytics Configuration
***********************

Analytics configurations allow creating **alerts**, configured individually for a critical parameter based on the following trigger types:

* High threshold value exceeded
* Low threshold value exceeded
* Return-to-midrange from High or Low threshold
* An relative change of value

Alerts
======

**Alerts** are configured via a ``config.dat`` file, manually via console commands, **OTA API** or **LSF API**, based on a simple criteria against a specific **parameter** defined by a ``paramId``.  

.. note::
	Only one **alert** can be defined against a given **paramId**, to avoid ambiguity of conditions.

In order to capture analog measurement values that may fluctuate within a small range, the minimum and maximum threshold values can be configured with bounds using on/off qualifiers.  
For example, consider a wind measurement in which we want an **alert** when the wind goes above a safe range of 45 km/h, but persist the alert condition until the wind drops below 38 km/h.
Assuming the Modbus register for wind speed is presented in tenths of km/h, we set maxON=450 and maxOFF=380.

Analytics have the following configurable attributes:

	* ``alertId`` (*required*)

		A unique identifier from **1..255** for a particular analytics configuration indicated in the **alert** message.

	* ``paramId`` (*required*)

		The unique **parameter** ID from **1..255** used as the **alert** trigger.  
		This parameter’s value will be included in the **alert** message by default.

	* ``paramIds`` (*optional*)

		A string of comma separated values of unique **parameter id(s)** that will be included in the alert, in addition to the trigger value.  
		This is typically used to cross-reference relevant data sources affected by the trigger value or as useful metadata to the alert condition.

	* ``minON`` (*optional*)

		A lower threshold value from **-2147483647..2147483647** against which the parameter value will be tested.
		If the parameter value drops *below minON*, a minimum ON alert :ref:`paramAlertON` will be generated, indicating that current value crossed the minimum threshold.  
		
		This alarm condition will persist until the value rises *above minOFF*.

	* ``minOFF`` (*optional*)

		A mid-range threshold value from **-2147483647..2147483647** against which the parameter value will be tested.  
		If the parameter value raises *above minOFF* after having dropped below minON, a minimum OFF alert :ref:`paramAlertOFF` will be generated, indicating that current value returned to the normal range.

	* ``maxON`` (*optional*)

		An upper threshold value from **-2147483647..2147483647** against which the parameter value will be tested. 
		If the parameter value raises *above maxON*, a maximum ON alert :ref:`paramAlertON` will be generated, indicating that current value crossed the maximum threshold.  

		This alarm condition will persist until the value drops *below maxOFF*.

	* ``maxOFF`` (*optional*)

		A mid-range threshold value from **-2147483647..2147483647** against which the parameter value will be tested. 
		If the parameter value drops *below maxOFF* after having raised above minON, a maximum OFF alert :ref:`paramAlertOFF` will be generated, indicating that current value returned to the normal range.

	* ``change`` (*optional*)

		A relative threshold value from **0..2147483647** against which the last reported parameter value will be tested.  
		If the parameter value changes by this value or more relative to the last reported value, an alert is generated.

	* ``msgBitmap`` (*optional*)
		
		An ASCII hex string value from **"00".."FF"** used to override the default Bitmap and include/exclude optional message fields in this particular **alert**.

		See: :ref:`msgBitmap <msgbitmap>`


Default Settings (Configuration Properties)
*******************************************

A **LSF** service contains configuration **properties** each referenced by unique Property Identification Number (**pin**) within the scope of the service (**sin**).

.. note::
	This section assumes the reader has a good understanding of the ORBCOMM Lua Service Framework and is familiar with how to read their Lua/Terminal API specification.


.. _geolocation-config:

Geolocation Movement Alert Configuration
========================================

By default, *geolocation* alerts are disabled.  To enable *geolocation* alerts, you must configure user service property ``assetMoveDist`` with a non-zero distance (in meters).
You can also change the user service property ``posAlarmDebounce`` from its default value to modify the *geolocation* behaviour.


LSF Properties
==============

.. _config-properties:

.. csv-table:: Configuration Properties
	:file: csvtables/config-properties.csv
	:header-rows: 1
	:widths: 5, 10, 20, 8, 7, 10


Optional Metadata Fields in Messages
====================================

Verbose metadata is, by default, enabled on all **reports** and **alerts**.  
You can change this behaviour by any of the following methods:

* Change the default for *ALL* **reports** and **alerts** using ``defaultMsgBitmap`` property value in your :ref:`build package <solution-studio-package>`
* Configure on a per-report or per-alert basis using your :ref:`configuration file <configfile>`
* or using the :ref:`OTA API <to-mobile-messages>`

.. _msgbitmap:

.. csv-table:: Optional Metadata Fields Configuration (MsgBitmap, defaultMsgBitmap)
	:file: csvtables/msgbitmap.csv
	:header-rows: 1
	:widths: 15, 7, 7, 7

