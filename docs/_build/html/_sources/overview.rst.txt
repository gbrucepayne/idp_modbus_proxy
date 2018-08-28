Overview
########

Concept of Operation
********************

The IDP/ST satellite terminal is typically configured using a simple **config file** loaded via its main RS232 serial port prior to field deployment, and connects to a specific Modbus device (e.g. PLC, RTU, etc.) or network using either RS232 or RS485.  The config file is set up based on mapping the Modbus device manufacturer’s register definitions to relevant parameters for the application.

The satellite terminal polls the attached Modbus device(s) registers at a frequent interval such as every 10 seconds, and sends the relevant data over the satellite network at a longer interval such as once per hour.  Register data gets mapped to **parameters** and consolidated into an efficient single over-the-air **report** message for satellite transmission.  In this way, the proxy **report** is much more data cost efficient than polling the device over-the-air.  **Reports** can also be defined to *not* transmit periodically, instead being queried by the enterprise/cloud application as a more efficient data set than polling individual contiguous register blocks (some people refer to this kind of operation as a *Modbus shift*).

Additionally, analytics are configurable on register data that can detect relevant changes or trigger values and generate an immediate **alert** containing associated data from multiple registers.  This method avoids the risk of missing critical events in between scheduled data transmissions, while minimizing network costs by avoiding more frequent polling/reporting.

An enterprise/cloud application uses a secure (RESTful) web service API to retrieve the messages, at a much lower cost than actively polling native Modbus registers over the satellite link.  The enterprise application decodes the register data based on the Modbus device-specific data values, and processes those accordingly.

.. figure:: figures/conop.jpg
	:scale: 100%
	:alt: concept of operation diagram

	IDP Modbus Proxy Concept of Operation


Software Architecture
*********************

The ``Modbus`` service is an embedded software application that acts as a data proxy between an industrial device and a centralized/cloud application.  It is designed to run on ORBCOMM IsatData Pro (IDP and ST series) terminals using the Lua Services Framework (**LSF**) as a *user service* with a default service identifier number **SIN 200**.

The cloud application retrieves data from and sends data to the Modbus service using Over-The-Air message operations (**OTA API**).

Although the Modbus service is intended for standalone operation, it also exports adequate means for integration with other LSF services (**LSF API**) to provide software programmers a means to implement various extensions to the core feature set.
The Modbus service exports to the **LSF** the following:
	
	* ``From-Mobile Messages`` describing reports, alerts and response operations performed *OTA*
	* ``To-Mobile Messages`` describing configuration and query operations performed *OTA*
	* ``Properties`` containing configuration and data accessible remotely or via Lua operations
	* ``Shell Commands`` operations that can be performed locally on the main serial port (if not in use for Modbus protocol)

.. note::
	The style of programming implementation for the various messages, events, and commands is case sensitive, and uses mixed case e.g. ``baudRate``.

.. warning::
	ORBCOMM IDP-series terminals (e.g. IDP-680, IDP-782) use a different version of Lua than ST-series terminals (e.g. ST6100).  The Modbus service has been designed to be compatible with both terminal hardware platforms, but requires the use of different package builds for each family type.  The service name is ``Modbus`` in both cases, but the package file uses a suffix either –IDP or –ST to indicate the target hardware platform.

.. note::
	Software programmers wishing to customize the service using their own Lua code, should consult with the relevant ORBCOMM product documentation for the target hardware platform (IDP series or ST series).
