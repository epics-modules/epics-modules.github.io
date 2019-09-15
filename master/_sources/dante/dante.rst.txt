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

These are the additional records that the Dante driver provides.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - CollectMode, CollectMode_RBV
     - mbbo, mbbi
     - DanteCollectMode
     - Controls the data collection mode.
       Choices are "MCA" (0), "MCA Mapping" (1) and "List Mapping" (2).
   * - CurrentPixel
     - longin
     - DanteCurrentPixel
     - The current pixel number in mapping modes.
   * - MaxEnergy, MaxEnergy_RBV
     - ao, ai
     - DanteMaxEnergy
     - The actual energy of the last channel.  The user must provide this value based on the energy calibration.
       It is used to provide meaningful units for the trigger thresholds, etc.
   * - PollTime, PollTime_RBV
     - ao, ai
     - DantePollTime
     - The time between polls when reading completion status from the driver.


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

