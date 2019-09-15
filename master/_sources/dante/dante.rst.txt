======================================
Dante
======================================

:author: Mark Rivers, University of Chicago

.. contents:: Contents

.. _dante:        https://github.com/epics-modules/dante
.. _mca:          https://github.com/epics-modules/mca
.. _asyn:         https://github.com/epics-modules/asyn
.. _asynNDArrayDriver: https://areadetector.github.io/master/ADCore/NDArray.html#asynndarraydriver
.. _XGLab:        https://www.xglab.it

Overview
--------

This is an EPICS driver for the XGLab_ Dante digital x-ray spectroscopy system.
The Dante is available in single channel and 8-channel versions
This module is intended to work with either, though it has currently only been tested on the single-channel version.

The Dante can collect data in 3 different modes:

- Single MCA spectrum.  It acquires a single MCA spectrum on all channels.
- MCA mapping mode.  It acquires multiple spectra in rapid succession, and it often used for making an x-ray map where there is an MCA
  spectrum for each channel at each pixel.  The advance to the next pixel can come from an internal clock or an external trigger.
- List mapping mode.  It acquires each x-ray event energy and timestamp in a list buffer.

The dante driver is derived from the base class asynNDArrayDriver_, which is part of the EPICS areaDetector package.
The allows the dante driver to use all of the areaDetector plugins for file saving in mapping mode and other purposes.


Dante driver
------------
Dante inherits from asynNDArrayDriver_.  It also implements the mca interface from the EPICS mca_ module.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - TimeStampMode, TimeStampMode_RBV
     - bo, bi
     - SP_TIME_STAMP_MODE
     - Controls whether the TimeStamp attribute comes from the camera internal time stamp or from the EPICS time.
       Choices are Camera (0), and EPICS (1).
   * - UniqueIdMode, UniqueIdMode_RBV
     - bo, bi
     - SP_UNIQUE_ID_MODE
     - Controls whether the UniqueId attribute comes from the camera internal value or from the driver.
       Choices are Camera (0), and Driver (1).
   * - ConvertPixelFormat, ConvertPixelFormat_RBV
     - mbbo, mbbi
     - SP_CONVERT_PIXEL_FORMAT
     - Controls conversion of the pixel format read from the camera to a different format.  For example this can be used
       to convert Mono12Packed to Mono16, which allows the camera to send 12-bit data over the bus and then convert to 16-bit
       on the host computer, reducing the required bandwidth and increasing the frame rate.
   * - FailedPacketCount
     - longin
     - SP_FAILED_PACKET_COUNT
     - Failed packet count
   * - FailedBufferCount
     - longin
     - SP_FAILED_BUFFER_COUNT
     - Failed buffer count
   * - BufferUnderrunCount
     - longin
     - SP_BUFFER_UNDERRUN_COUNT
     - Buffer underrun count


IOC startup script
------------------
The command to configure an ADSpinnaker camera in the startup script is::

  ADSpinnakerConfig(const char *portName, const char *cameraId, int traceMask, int memoryChannel,
                    size_t maxMemory, int priority, int stackSize)

``portName`` is the name for the ADSpinnaker port driver

``cameraId`` is the either the serial number of the camera or the camera index number in the system.  The serial number is normally printed
on the camera, and it is also the last part of the camera name returned by arv-tool, for example for
``"Point Grey Research-Blackfly S BFS-PGE-50S5C-18585624"``, it would be 18585624. 
If cameraId is less than 1000 it is assumed to be the system index number, if 1000 or greater it is assumed to be a serial number.

``traceMask`` is the initial value of asynTraceMask to be used for debugging problems in the constructor.

``memoryChannel`` is the internal channel number in the camera to be used for saved cameras settings.

``maxMemory`` is the maximum amount of memory the NDArrayPool is allowed to allocate.  0 means unlimited.

``priority`` is the priority of the port thread.  0 means medium priority.

``stackSize`` is the stack size.  0 means medium size.

MEDM screens
------------
The following is the main MEDM screen dante.adl.

.. figure:: dante.png
    :align: center

The following is the MEDM screen mca.adl displaying the mca spectrum as it is acquiring.

.. figure:: dante_mca.png
    :align: center

The following is the MEDM screen danteTrace.adl displaying the ADC trace. One reset is visible on this trace.

.. figure:: dante_trace.png
    :align: center

The following is the MEDM screen NDFileHDF5.adl when the Dante is saving MCA mapping data to an HDF5 file.

.. figure:: dante_mapping_hdf5.png
    :align: center

