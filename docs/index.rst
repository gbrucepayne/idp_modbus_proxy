.. IDP Modbus Proxy documentation
   Copyright: Inmarsat 2018
   The `toctree` directive is from Sphinx, a custom directive allowing linkage
   of multiple documents.
   Should also include term role and glossary directive

.. |logo1| image:: images/inmarsat.jpg
	:width: 200px
	:align: middle
	:height: 78px
	:alt: Inmarsat

.. |logo2| image:: images/gse.jpg
	:width: 250px
	:align: middle
	:height: 94px
	:alt: Global Satellite Engineering

.. table::
	:align: center

	+---------+---------+
	| |logo1| | |logo2| |
	+---------+---------+

.. |C| unicode:: 0xA9
	:ltrim:

.. |TM| unicode:: U+2122
	:ltrim:

.. |R| unicode:: U+00AE
	:ltrim:

################################
 Modbus Proxy for IDP Terminals
################################

:Date: 2018-08-20
:Revision: 1
:Authors:
	- Geoff Bruce-Payne
	- Yanosh Sabo
:Contributors:
	- Robert Gauthier
	- Andrew Windridge

Introduction
############

`Modbus <http://www.modbus.org>`_ is a widely used industrial automation data protocol based on a Master/Slave architecture, often used in Supervisory Control and Data Acquisition (SCADA) systems.
Many industrial automation devices use Modbus protocol over a serial interface (RS-232 or RS-485).

IsatData Pro |R| is a *highly-reliable, global* satellite data service offered by `Inmarsat <https://www.inmarsat.com/service/isatdata-pro/>`_, well suited to transporting small amounts of machine-to-machine (M2M) and *Industrial Internet of Things* (IIoT) data to and from industrial devices (e.g. Modbus) located in remote locations.

The Modbus Proxy service is an *embedded* software application written in the `Lua <https://www.lua.org>`_ programming language that runs on certain versions of IsatData Pro compatible satellite terminals manufactured by `ORBCOMM <https://www.orbcomm.com/en/hardware/devices/st-series>`_.

.. admonition:: Disclaimer
	
	While the information in this document has been prepared in good faith, no representation, warranty, assurance or undertaking (express or implied) is or will be made, and no responsibility or liability (howsoever arising) is or will be accepted by the Inmarsat group or any of its officers, employees or agents in relation to the adequacy, accuracy, completeness, reasonableness or fitness for purpose of the information in this document. All and any such responsibility and liability is expressly disclaimed and excluded to the maximum extent permitted by applicable law. INMARSAT is a trademark owned by the International Mobile Satellite Organisation, the Inmarsat LOGO is a trademark owned by Inmarsat (IP) Company Limited. Both trademarks are licensed to Inmarsat Global Limited. All other Inmarsat trade marks in this document are owned by Inmarsat Global Limited.

Contents
########

.. toctree::
	:numbered:

	overview
	funcspec
	usage
	configuration
	otaapi
	lsfapi
	testing

.. documents can be listed here, indented under toctree

.. ignore what's below, style reference

	.. _test-label:

	Cross-reference to :ref:`label <test-label>` above.

	Indices and tables (H1)
	#######################

	* :ref:`genindex`
	* :ref:`modindex`
	* :ref:`search`

	Sample H2
	*********

	Sample H3
	=========

	Sample H4
	---------

	Sample H5
	^^^^^^^^^

	Sample H6
	"""""""""

	Some more text.