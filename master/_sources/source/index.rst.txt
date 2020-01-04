=====
Dante
=====

:author: Mark Rivers, University of Chicago

.. contents:: Contents

.. _dante:             https://github.com/epics-modules/dante
.. _mca:               https://github.com/epics-modules/mca
.. _asyn:              https://github.com/epics-modules/asyn
.. _asynNDArrayDriver: https://areadetector.github.io/master/ADCore/NDArray.html#asynndarraydriver
.. _areaDetector:      https://areadetector.github.io
.. _XGLab:             https://www.xglab.it

Overview
--------

This is an EPICS driver for the XGLab_ Dante digital x-ray spectroscopy system.
The Dante is available in single channel and 8-channel versions
This module is intended to work with either, though it has currently only been tested on the single-channel version.
In this document NumBoards refers to the number of input channels, e.g 1 for a single-channel Dante and 8 for an 
8-channel Dante.

The Dante can collect data in 3 different modes:

- Single MCA spectrum.  It acquires a single MCA spectrum on all channels.
- MCA mapping mode.  It acquires multiple spectra in rapid succession, and it often used for making an x-ray map where there is an MCA
  spectrum for each channel at each pixel.  The advance to the next pixel can come from an internal clock or an external trigger.
- List mapping mode.  It acquires each x-ray event energy and timestamp in a list buffer.

The Dante driver is derived from the base class asynNDArrayDriver_, which is part of the EPICS areaDetector_ package.
The allows the Dante driver to use all of the areaDetector plugins for file saving in MCA mapping and list modes,
and for other purposes. It also implements the mca interface from the EPICS mca_ module.
The EPICS mca record can be used to display the spectra and control the basic operation including Regions-of-Interest (ROIs).

The Dante driver can be used on both Windows and Linux. A Windows machine with a USB interface is currently required
to load new firmware.  Otherwise the module can be used from either Linux or Windows over Ethernet. Currently XGLab only
provides the Linux library built with the version of gcc on Ubuntu 18.  This means it cannot be used on RHEL7 or Centos7, for
example.  However, they are planning to release a build done with an older compiler that with run on RHEL7/Centos7 soon.

This document does not attempt to give an explanation of the principles of operation of the Dante, or a detailed explanation
of the many configuration parameters for the digital pulse processing.  The user should consult the
:download:`DanteManual <DANTE-4552 Manual Rev 2.9.pdf>` for this information.

System controls
---------------
These records are in the file ``dante.template``. This database is loaded once for the Dante system.  It provides
control of the system-wide settings for the system.

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
       Choices are "MCA" (0), "MCA Mapping" (1) and "List" (2).
   * - GatingMode, GatingMode_RBV
     - mbbo, mbbi
     - DanteGatingMode
     - Controls the gating mode.
       Choices are "Free running" (0), "Trig rising" (1), "Trig falling" (2), "Trig both" (3), "Gate high" (4), "Gate low" (5).
   * - NumMCAChannels, NumMCAChannels_RBV
     - mbbo, mbbi
     - MCA_NUM_CHANNELS
     - The number of MCA channels to use.  Choices are 1024, 2048, 4096.
   * - PollTime, PollTime_RBV
     - ao, ai
     - DantePollTime
     - The time between polls when reading completion status, MCA mapping data, and list mode data from the driver.
       0.01 second is a reasonable value that will provide good response and resource utilization.
   * - PresetReal
     - ao
     - MCA_PRESET_REAL
     - Sets the preset real time.  Set this to 0 to count forever in MCA mode or List mode.
   * - EraseStart
     - bo
     - N.A.
     - Processing this record starts acquisition for all boards in the selected CollectMode.
   * - StartAll
     - bo
     - MCA_START_ACQUIRE
     - Processing this record starts acquisition for all boards in the selected CollectMode. This record should not
       be used by higher-level software, it is processed by EraseStart.
   * - MCAAcquireBusy
     - busy
     - N.A.
     - This record goes to 1 ("Collecting") when EraseStart is processed. It goes back to 0 ("Done")when 3 conditions
       are satisfied. 1) MCAAcquiring is 0; 2) All MCA records have .ACQG field=0; 3)AcquireBusy from areaDetector=0.
       The last condition can ensure that all plugins are done processing if WaitForPlugins is set.
   * - MCAAcquiring
     - bi
     - MCA_ACQUIRING
     - This record is 1 when the Dante driver itself is acquiring, and 0 when it is done. This record is generally not used
       by higher level software, use MCAAcquireBusy instead, since it indicates when all components are done.
   * - StopAll
     - bo
     - MCA_STOP_ACQUIRE
     - Processing this record stops acquisition for all boards in the selected CollectMode. This only needs to be used
       to terminate acquisition before it would otherwise stop because PresetReal or NumMappingPoints have been reached.
   * - ReadAll
     - bo
     - N.A.
     - Processing this record reads the MCA data and statistics for all boards.  This .SCAN field of this record is typically
       set to periodic, i.e. "1 second", ".1 second", etc. to provide user feedback while acquisition is in progress.
       It can be set to "Passive" and the system will still read the data once when acquisition completes. 
       This can be used to improve performance at very short PresetReal times. 
       This record is disabled when acquisition is complete to reduce unneeded resource usage.
   * - ReadAllOnce
     - bo
     - N.A.
     - Processing this record reads the MCA data and statistics for all boards.  This record is processed by ReadAll. It can be
       manually processed to read the data even when acquisition is complete.
   * - ElapsedReal
     - ai
     - MCA_ELAPSED_REAL
     - The elapsed real time.
   * - ElapsedLive
     - ai
     - MCA_ELAPSED_LIVE
     - The elapsed live time.
   * - DeadTime
     - ai
     - DanteDeadTime
     - The cummulative deadtime.
   * - IDeadTime
     - ai
     - DanteIDeadTime
     - The "instantaneous" deadtime since the previous readout.
          

Configuration parameters
------------------------
These records control the configuration of the digital signal processing. The readback (_RBV) values may differ slightly
from the output values because of the discrete nature of the system clocks and MCA bins.

These parameters are specific to a single board, and are contained in DanteN.template.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - MaxEnergy, MaxEnergy_RBV
     - ao, ai
     - DanteMaxEnergy
     - The actual energy of the last channel.  The user must provide this value based on the energy calibration.
       It is used to provide meaningful units for FastThreshold, EnergyThreshold, and BaselineThreshold.
   * - InputPolarity, InputPolarity_RBV
     - bo, bi
     - DanteInvertedInput
     - The pre-amp output polarity. Choices are "Pos." (0) and "Neg." (1).
   * - AnalogOffset, AnalogOffset_RBV
     - longout, longin
     - DanteAnalogOffset
     - The analog offset applied to the input signal, 0 to 255. 
       This offset must be adjusted to keep the input signal within the range of the ADC.
       This should be adjusted using the ADC Trace plot with a long sampling to see the range of the input
       signal through a reset event.
   * - ResetThreshold, ResetThreshold_RBV
     - longout, longin
     - DanteResetThreshold
     - The reset threshold in ADC units per N 8 ns sample intervals. The Dante detects a reset the signal changes by more than this amount. 
       The standard firmware uses N=6 and this ResetThreshold value.
       The high-rate firmware uses N=1 and fixes ResetThreshold=256, so this parameter has no effect.
   * - ResetRecoveryTime, ResetRecoveryTime_RBV
     - ao, ai
     - DanteResetRecoveryTime
     - The time in microseconds to wait after a reset event.
   * - Gain, Gain_RBV
     - ao, ai
     - DanteGain
     - The gain which controls the number of ADC units per MCA bin.  Gains of 1.0-4.0 are typical.
   * - FastThreshold, FastThreshold_RBV
     - ao, ai
     - DanteFastFilterThreshold
     - The fast filter threshold in keV.
   * - FastPeakingTime, FastPeakingTime_RBV
     - ao, ai
     - DanteEdgePeakingTime
     - The peaking time of the fast filter in microseconds.
   * - FastFlatTopTime, FastFlatTopTime_RBV
     - ao, ai
     - DanteEdgeFlatTop
     - The flat top time of the fast filter in microseconds.
   * - EnergyThreshold, EnergyThreshold_RBV
     - ao, ai
     - DanteEnergyFilterThreshold
     - The energy filter threshold in keV.
   * - PeakingTime, PeakingTime_RBV
     - ao, ai
     - DantePeakingTime
     - The peaking time of the slow filter in microseconds.
   * - MaxPeakingTime, MaxPeakingTime_RBV
     - ao, ai
     - DanteMaxPeakingTime
     - The maximum peaking time of the slow filter in microseconds. Used only with the high-rate firmware.
       Must be set to 0 when using the standard firmware.
   * - FlatTopTime, FlatTopTime_RBV
     - ao, ai
     - DanteFlatTop
     - The flat top time of the slow filter in microseconds.
   * - BaselineThreshold, BaselineThreshold_RBV
     - ao, ai
     - DanteEnergyBaselineThreshold
     - The baseline filter threshold in keV.
   * - MaxRiseTime, MaxRiseTime_RBV
     - ao, ai
     - DanteMaxRiseTime
     - The maximum rise time in usec. Pulses with a longer rise time will be pileup rejected.
   * - ZeroPeakFreq, ZeroPeakFreq_RBV
     - ao, ai
     - DanteZeroPeakFreq
     - The frequency of the zero-energy peak in Hz.
   * - BaselineSamples, BaselineSamples_RBV
     - longout, longin
     - DanteBaselineSamples
     - The number of baseline samples.  Typical value is 64.
   * - TimeConstant, TimeConstant_RBV
     - ao, ai
     - DanteTimeConstant
     - The time constant. Used for digital deconvolution in the case of continuous reset signals.
   * - TailCoefficient, TailCoefficient_RBV
     - ao, ai
     - DanteTailCoefficient
     - The tail coefficient. Not currently used.
   * - BaseOffset, BaseOffset_RBV
     - longout, longin
     - DanteBaseOffset
     - The base offset. Used for digital deconvolution in the case of continuous reset signals.
   * - OverflowRecoveryTime, OverflowRecoveryTime_RBV
     - ao, ai
     - DanteOverflowRecoveryTime
     - The overflow recovery time. Not currently used.

Run-time statistics
-------------------
These are the records for run-time statistics.

These parameters are specific to a single board, and are contained in DanteN.template.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - InputCountRate
     - ai
     - DanteInputCountRate
     - The input count rate in kHz.
   * - OutputCountRate
     - ai
     - DanteOutputCountRate
     - The output count rate in kHz.
   * - Triggers
     - longin
     - DanteTriggers
     - The number of triggers received.
   * - Events
     - longin
     - DanteEvents
     - The number of events received.
   * - FastDeadTime
     - longin
     - DanteEdgeDTime
     - The fast deadtime in clock ticks.
   * - F1DeadTime
     - longin
     - DanteFilt1DT
     - The filter 1 deadtime in clock ticks.
   * - ZeroCounts
     - longin
     - DanteZeroCounts
     - The number of zero count events.
   * - BaselineCount
     - longin
     - DanteBaselinesValue
     - The number of baseline events.
   * - PileUp
     - longin
     - DantePUPValue
     - The number of pileup events.
   * - F1PileUp
     - longin
     - DantePUPF1Value
     - The number of filter 1 pileup events.
   * - NotF1PileUp
     - longin
     - DantePUPNotF1Value
     - The number of not filter 1 pileup events.
   * - ResetCounts
     - longin
     - DanteResetCounterValue
     - The number of reset events.
   * - LastTimeStamp
     - ai
     - DanteLastTimeStamp
     - The last timestamp time in clock ticks.

The following is the main MEDM screen dante.adl. This screen is used with the 1-channel Dante.  Multi-board Dante systems will
use a different screen that has not yet been created.

.. figure:: dante.png
    :align: center

MCA mode
--------
The MCA mode collects a single MCA record at a time.  It is compatible with the MCA record, and is the same
as MCA operation on many other EPICS MCAs, e.g. Canberra AIM, Amptek, XIA (Saturn, Mercury, xMAP, FalconX), SIS38XX, and others.

It only supports counting for a preset real time, or counting indefinitely (PresetReal=0).
It does not support PresetLive or PresetCounts which some other MCAs do.

The following is the MEDM screen mca.adl displaying the MCA spectrum as it is acquiring.

.. figure:: dante_mca.png
    :align: center

The following is the IDL MCA Display program showing the MCA spectrum as it is acquiring. This GUI allows defining ROIs
graphically, fitting peaks and background, and many other features.

.. figure:: dante_idl_mca.png
    :align: center

MCA mapping mode
----------------
These are the records for MCA Mapping mode.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - CurrentPixel
     - longin
     - DanteCurrentPixel
     - In MCA Mapping mode this is the current pixel number.  In List mode it is the total number of x-ray events received so far.
   * - MappingPoints, MappingPoints_RBV
     - longout, longin
     - DanteMappingPoints
     - The number of spectra to collect in MCA mapping mode.
     
In MCA mapping mode the GatingMode can be "Free running", "Trig rising", "Trig falling", or "Trig both".
In free-running mode the Dante will begin the next spectrum when the PresetReal time has elapsed.
In triggered mode the Dante will begin the next spectrum when the when a trigger occurs 
or when the PresetReal time has elapsed, whichever comes first.
To advance only on trigger events set the PresetReal time to a value larger than the maximum time between triggers.

The MCA spectra are copied into NDArrays of dimensions [NumMCAChannels, NumBoards]. For a 1-channel Dante
NumBoards is 1.  The run-time statistics for each spectrum are copied into NDAttributes attached to each
NDArray. The attribute names contain the board number, for example "RealTime_0".

The NDArrays can be used by any of the standard areaDetector plugins.  For example, they can be streamed
to HDF5, netCDF, or TIFF files.

The following is the MEDM screen NDFileHDF5.adl when the Dante is saving MCA mapping data to an HDF5 file.

.. figure:: dante_mapping_hdf5.png
    :align: center


List mode
---------
These are the records for list mode.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - CurrentPixel
     - longin
     - DanteCurrentPixel
     - In List mode this is the total number of x-ray events received so far.
   * - ListBufferSize, ListBufferSize_RBV
     - longout, longin
     - DanteListBufferSize
     - The number of x-ray events per buffer in list mode. 
       Once this number of events has been received the events read from the Dante
       stored in NDArrays, and callbacks are done to any registered plugins.

List mode events are 64-bit unsigned integers.

- Bits 0 to 15 are the x-ray energy, i.e. ADC value.
- Bits 16 to 17 are not used.
- Bits 18 to 61 are the timestamp in 8 ns units.
- Bits 62 and 63 are not used.

In list mode the x-ray events are copied into NDArrays.
Because the EPICS asyn and areaDetector modules do not yet support 64-bit integers the data type of
the NDArrays is set to NDUInt8, and the NDArrayDimensions are [ListBufferSize*8, NumBoards].
For a 1-channel Dante NumBoards is 1.

The run-time statistics for ListBufferSize events are copied into NDAttributes attached to each
NDArray. The attribute names contain the board number, for example "RealTime_0".
Note that these statistics are cummulative for the entire acquisition, not just since the
last time the event buffer was read.
By making ListBufferSize smaller one obtains a more frequent sampling of these statistics.

These statistics also update the run-time statistics records described above, so there is feedback
while the list mode acquisition is in progress.

The first NumMCAChannels events are copied to the buffer for the MCA record for each board.
In this case the MCA record will not contain an x-ray spectrum, but rather will contain the x-ray
energy in ADC units on the vertical axis and the event number on the horizontal axis.

The NDArrays can be used by any of the standard areaDetector plugins.  For example, they can be streamed
to HDF5, netCDF, or TIFF files.

Note that the datatype in the files is unsigned 8-bit integers.  Applications that read the arrays must
cast them to unsigned 64-bit arrays before operating on them.
In the future support for 64-bit integers will be added to asyn and areaDetector, and the NDArrays will
have the correct new NDUInt64 datatype.

The following is an IDL procedure to read the List mode data from a netCDF file into two arrays, "energy" and "time"::

  pro read_dante_list_data, filename, energy, time
     raw = read_nd_netcdf(filename)
     data = ulong64(raw, 0, n_elements(raw)/8)
     energy = uint(data and 'ffff'x)
     time = double(ishft((data and '3ffffffffffc0000'x), -18))*8e-9
  end

``read_nd_netcdf`` is a function provide in the areaDetector_ package that reads a netCDF file written by the areaDetector
NDFileNetCDF plugin.
The following is a plot of the energy events for the first 1 second of that data, using this IDL command::

  IDL> p = plot(time, energy, xrange=[0,1], yrange=[0,20000], linestyle='none', symbol='plus')

.. figure:: dante_idl_list_plot.png
    :align: center

ADC trace waveforms
-------------------
The Dante can collect ADC trace waveforms, which is effectively a digital oscilloscope of the pre-amp input signal.
This very useful for setting the AnalogOffset record, and for diagnosing issues with the input.

These are the records to control ADC traces. All of the records except TraceData affect all boards and are in dante.template.
TraceData is specific to each board and is in danteN.template.

.. cssclass:: table-bordered table-striped table-hover
.. list-table::
   :header-rows: 1
   :widths: auto

   * - EPICS record names
     - Record types
     - drvInfo string
     - Description
   * - TraceTimeArray
     - waveform
     - DanteTraceTimeArray
     - Waveform record containing the time values for each point in TraceData. 64-bit float data type.
   * - TraceTime, TraceTime_RBV
     - ao, ai
     - DanteTraceTime
     - Time per sample of the ADC trace data in microseconds. Allowed range is 0.016 to 0.512.
   * - TraceLength, TraceLength_RBV
     - longout, longin
     - DanteTraceLength
     - The number of samples to read in the ADC trace.  This must be a multiple of 16384, and will be limited by the 
       NELM field of the TraceData and TraceTimeArray waveform records.
   * - TraceTriggerLevel, TraceTriggerLevel_RBV
     - longout, longin
     - DanteTraceTriggerLevel
     - The trigger level in ADC units (0 to 65535).
   * - TraceTriggerRising, TraceTriggerRising_RBV
     - bo, bi
     - DanteTraceTriggerRising
     - Trigger the ADC trace as it rises through TraceTriggerLevel. Choices are "No" (0) and "Yes" (1).
   * - TraceTriggerFalling, TraceTriggerFalling_RBV
     - bo, bi
     - DanteTraceTriggerFalling
     - Trigger the ADC trace as it fals through TraceTriggerLevel. Choices are "No" (0) and "Yes" (1).
   * - TraceTriggerInstant, TraceTriggerInstant_RBV
     - bo, bi
     - DanteTraceTriggerInstant
     - Trigger the ADC trace even if a rising or falling trigger is not detected. Choices are "No" (0) and "Yes" (1).
   * - TraceTriggerWait, TraceTriggerWait_RBV
     - ao, ai
     - DanteTraceTriggerWait
     - The delay time after the trigger condition is satisfied before beginning the ADC trace.
   * - TraceData
     - waveform
     - DanteTraceData
     - Waveform record containing the ADC trace data. 32-bit integer data type.

The following is the MEDM screen danteTrace.adl displaying the ADC trace. One reset is visible on this trace.
This happens to be from a Vortex SDD detector with a pre-amp ramp range that is slightly larger than this
Dante was factory-configured to use.  Thus the signal goes above and below the range of the ADC around the
reset.  The total pre-amp voltage range must be specified when ordering the Dante so that the signal will
stay in the range of the ADC.

.. figure:: dante_trace.png
    :align: center

     
IOC startup script
------------------
The command to configure an ADSpinnaker camera in the startup script is::

  DanteConfig(portName, ipAddress, numDetectors, maxMemory)

``portName`` is the name for the Dante port driver

``ipAddress`` is the IP address of the Dante 

``numDetectors`` is the number of boards in the Dante system

``maxMemory`` is the maximum amount of memory the NDArrayPool is allowed to allocate.  0 means unlimited.


Known problems
--------------
The known problems and suggestions for fixes with the Dante are located here:

https://onedrive.live.com/view.aspx?resid=B1EF6D9CFBC508F4!621&ithint=file%2cdocx&authkey=!AFZfJ0wJfFbDCAI



  