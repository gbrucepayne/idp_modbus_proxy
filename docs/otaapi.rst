OTA API Integration
###################


The over-the-air interface uses **messages** as the basis for communication.  
Messages are retrieved and submitted using Inmarsat's `IDP Messaging API <https://developer.inmarsat.com/content/isatdatapro-messaging-api>`_.

The key Messaging API operations used are:

* ``get_return_messages`` is used for retrieving periodic **reports**, **alerts** and parameter/data read responses.  See :ref:`from-mobile-messages`.
* ``submit_messages`` is used for configuration, and on-demand reading/writing of **parameters** or raw Modbus data.  See :ref:`to-mobile-messages`.

Message Structure
*****************

Each message is structured as a set of data **fields** that encapsulate different types of data.

**Return Messages** (aka Mobile-Originated, From-Mobile) are structured as follows::

   Messages[{
      ID,
      MessageUTC,
      ReceiveUTC,
      SIN,
      MobileID,
      OTAMessageSize,
      RawPayload,
      RegionName,
      Payload{
         Name,
         SIN,
         MIN,
         Fields[
            {
            Name,
            Type,
            <Value>,
            <Elements>[
               {
               Index,
               Fields[
                  {
                  Name,
                  Type,
                  Value
                  }]
               }]
            }
         ]
      }
   }]

**Forward Messages** (aka Mobile-Terminated, To-Mobile) are structured as follows::

   Messages[{
      DestinationID,
      <UserMessageID>,
      <RawPayload>,
      <Payload>[
         <Name>,
         <isForward>,
         SIN,
         MIN,
         <Fields>[
            {
            Name,
            <Type>,
            <Value>,
            <Elements>[
               {
               Index,
               Fields[
                  {
                  Name,
                  <Type>,
                  Value
                  }]
               }]
            }
         ]
      ]
   }]


Example
=======

An **report** with a single **parameter** ``Payload`` within a ``ReturnMessage`` for :ref:`periodicReport` is::
	
	Payload[
	  Name: "periodicReport",
	  SIN: 200,
	  MIN: 100,
	  Fields: [
		{
		  Name: "reportId",
		  Type: "unsignedint",
		  Value: 1
		},
		{
		  Name: "parameters",
		  Type: "array",
		  Elements: [
			{
			  Index: 0,
			  Fields: [
				{
				  Name: "paramId",
				  Type: "unsignedint",
				  Value: 1
				},
				{
				  Name: "value",
				  Type: "unsignedint",
				  Value: 300
				}
			  ]
			}
		  ]
		}
	  ]
	]


Message Definitions
*******************

.. admonition:: How to Read the Tables

	The tables used in this section follow the convention of the ORBCOMM Lua Terminal API documentation (**T202**, **T405**).

	* **Field Name** is a description for the ``Field.Name`` *(case sensitive)*
	* **Type** is the (data) type passed in the REST API

		* ``boolean`` for values held in Coils or Digital Inputs
		* ``unsignedint`` is used for 16-bit unsigned integers
		* ``signedint`` is used for both 16-bit and 32-bit signed integers
		* ``string`` is used to transport 32-bit floating point numbers and ASCII data
		* ``data`` is used to transport **raw** byte arrays
		* ``array`` is used to encapsulate **parameters**
		* ``enum`` is used by some configuration messages to enumerate static labels with integer values

	* **Optional** means that the field may or may not be included in a submission or response
	* **Fixed** means that the length of the field is fixed *(vs. variable)*
	* **Size** is the length of the field, depending on the Type encoding it represents:

		* *Bits* when the **Type** is ``unsignedint``, ``signedint``, ``boolean``, or ``enum``
		* *Bytes* when the **Type** is ``string`` *(characters)* or ``data``
		* *Elements (maximum)* when the **Type** is ``array`` *(note that array OTA size is complex to calculate)*

.. note::
	The Lua Service Framework used on IDP-series terminals does not support unsigned 32-bit integers, so the Modbus service only supports 16-bit unsigned.
	You could work around this by defining a 32-bit unsignedint as ``raw`` in ``config.dat``.

.. note::
	The Lua Service Framework does not support floating point data types natively as message fields.  
	Modbus service converts IEEE floating point to a string value of the decimal number *(e.g. "123.456" instead of 0x42f6e979 or [66 246 233 121])*.


.. _to-mobile-messages:

To-Mobile Messages
==================

.. csv-table:: To-Mobile Messages
	:file: csvtables/to-mobile-messages.csv
	:header-rows: 1
	:widths: 10, 25, 70

------

.. _setModbusConfig:

setModbusConfig (MIN 10)
------------------------

Configures Modbus **ports**, **devices**, **parameters** (and optionally **properties**).

.. csv-table:: setModbusConfig
	:file: csvtables/setmodbusconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _getModbusConfig:

getModbusConfig (MIN 11)
------------------------

Retrieves select Modbus configuration(s).

.. csv-table:: getModbusConfig
	:file: csvtables/getmodbusconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _delModbusConfig:

delModbusConfig (MIN 12)
------------------------

Deletes select Modbus configuration(s).

.. csv-table:: delModbusConfig
	:file: csvtables/delmodbusconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _setReportConfig:

setReportConfig (MIN 13)
------------------------

Configures periodic **reports**.

.. csv-table:: setReportConfig
	:file: csvtables/setreportconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _getReportConfig:

getReportConfig (MIN 14)
------------------------

Retrieves select Report configuration(s).

.. csv-table:: getReportConfig
	:file: csvtables/getreportconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _delReportConfig:

delReportConfig (MIN 15)
------------------------

Deletes select Report configuration(s).

.. csv-table:: delReportConfig
	:file: csvtables/delreportconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _setAlertConfig:

setAlertConfig (MIN 16)
-----------------------

Configures **alerts**.

.. csv-table:: setAlertConfig
	:file: csvtables/setalertconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _getAlertConfig:

getAlertConfig (MIN 17)
-----------------------

Retrieves select Alert configuration(s).

.. csv-table:: getAlertConfig
	:file: csvtables/getalertconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _delAlertConfig:

delAlertConfig (MIN 18)
-----------------------

Deletes select Alert configuration(s).

.. csv-table:: delAlertConfig
	:file: csvtables/delalertconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _readParam:

readParam (MIN 19)
------------------

Reads select **parameter** values immediately.

.. csv-table:: readParam
	:file: csvtables/readparam.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _writeParam:

writeParam (MIN 20)
-------------------

Writes select **parameter** values immediately.  (Must be ``coil`` or ``holding`` types)

.. csv-table:: writeParam
	:file: csvtables/writeparam.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _readData:

readData (MIN 21)
-----------------

Reads select blocks of Modbus addresses directly.

.. csv-table:: readData
	:file: csvtables/readdata.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _writeData:

writeData (MIN 22)
------------------

Writes select blocks of Modbus addresses directly.  (Must be *Output Coils* or *Holding Registers*)

.. csv-table:: writeData
	:file: csvtables/writedata.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _setRefPosition:

setRefPosition (MIN 23)
--------------------------

Acknowledges the receipt of a :ref:`positionChangeAlert` and sets the current location as the new movement reference.

This message contains no **Fields**, it is just submitted with **SIN** and **MIN**.


.. _from-mobile-messages:

From-Mobile Messages
====================

.. csv-table:: From-Mobile Messages
	:file: csvtables/from-mobile-messages.csv
	:header-rows: 1
	:widths: 10, 25, 70

------

.. _modbusConfig:

modbusConfig (MIN 10)
----------------------

Response to :ref:`setModbusConfig` or :ref:`getModbusConfig` or :ref:`delModbusConfig`.

.. csv-table:: modbusConfig
	:file: csvtables/modbusconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _reportConfig:

reportConfig (MIN 11)
-------------------------

Response to :ref:`setReportConfig` or :ref:`getReportConfig` or :ref:`delReportConfig`.

.. csv-table:: reportConfig
	:file: csvtables/reportconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _alertConfig:

alertConfig (MIN 12)
-------------------------

Response to :ref:`setAlertConfig` or :ref:`getAlertConfig` or :ref:`delAlertConfig`.

.. csv-table:: alertConfig
	:file: csvtables/alertconfig.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _readParamResults:

readParamResults (MIN 13)
-------------------------

Response to :ref:`readParam`.

.. csv-table:: readParamResults
	:file: csvtables/readparamresults.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _writeParamResults:

writeParamResults (MIN 14)
--------------------------

Response to :ref:`writeParam`.

.. csv-table:: writeParamResults
	:file: csvtables/writeparamresults.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _readDataResults:

readDataResults (MIN 15)
------------------------

Response to :ref:`readData`.

.. csv-table:: readDataResults
	:file: csvtables/readdataresults.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _writeDataResults:

writeDataResults (MIN 16)
-------------------------

Response to :ref:`writeData`.

.. csv-table:: writeDataResults
	:file: csvtables/writedataresults.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _periodicReport:

periodicReport (MIN 100)
------------------------

Sent automatically at the prescribed time interval or offset from Time of Day.

.. csv-table:: periodicReport
	:file: csvtables/periodicreport.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _paramAlertON:

paramAlertON (MIN 101)
----------------------

Sent automatically when an **alert** condition has been triggered and is outside normal operating range.

.. csv-table:: paramAlertON
	:file: csvtables/paramalerton.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _paramAlertOFF:

paramAlertOFF (MIN 102)
-----------------------

Sent automatically when an **alert** condition has returned to the normal operating range.

.. csv-table:: paramAlertOFF
	:file: csvtables/paramalertoff.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _paramAlertChange:

paramAlertChange (MIN 103)
--------------------------

Sent automatically when an **alert** condition has changed from the last reported value.

.. csv-table:: paramAlertChange
	:file: csvtables/paramalertchange.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _positionChangeAlert:

positionChangeAlert (MIN 104)
-----------------------------

Sent automatically when movement has been detected and *geolocation* alerts are configured.

.. csv-table:: positionChangeAlert
	:file: csvtables/positionchangealert.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

.. _error:

error (MIN 200)
---------------

Sent automatically when a Modbus communication or configuration problem is detected.

.. csv-table:: error
	:file: csvtables/error.csv
	:header-rows: 1
	:widths: 25, 10, 5, 5, 5, 50

