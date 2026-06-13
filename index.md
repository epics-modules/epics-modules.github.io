---
layout: default
title: Home
nav_order: 1
---

# EPICS Modules

The [epics-modules](https://github.com/epics-modules) GitHub organization is a
community-maintained collection of support modules for
[EPICS](https://epics-controls.org/) (Experimental Physics and Industrial
Control System). These modules extend EPICS with device drivers, record types,
communication protocols, and utilities used at research facilities worldwide.

Many of these modules originated as part of
[synApps](https://www.aps.anl.gov/BCDA/synApps), a collection of EPICS software
developed primarily by the BCDA group at Argonne National Laboratory.

---

## Modules

### Communication & Driver Support

| Module | Description |
|--------|-------------|
| [asyn](https://epics-modules.github.io/asyn/) | Driver and device support for asynchronous communication (serial, TCP/IP, GPIB, and more) |
| [modbus](https://epics-modules.github.io/modbus/) | Communication with PLCs via Modbus protocol over TCP, serial RTU, and serial ASCII |
| [stream](https://github.com/epics-modules/stream) | StreamDevice protocol-based communication (archived -- see [StreamDevice](https://paulscherrerinstitute.github.io/StreamDevice/)) |
| [ether_ip](https://github.com/epics-modules/ether_ip) | EtherNet/IP support for Allen Bradley PLCs |
| [mqtt](https://github.com/epics-modules/mqtt) | MQTT protocol support based on asyn and Paho |
| [twincat-ads](https://github.com/epics-modules/twincat-ads) | Communication with Beckhoff TwinCAT controllers over ADS protocol |
| [opcua](https://github.com/epics-modules/opcua) | OPC UA device support |

### Motor & Motion Control

| Module | Description |
|--------|-------------|
| [motor](https://epics-modules.github.io/motor/) | Motor record and driver support for motion controllers |
| [tpmac](https://github.com/epics-modules/tpmac) | Support for Delta Tau Turbo PMAC2 motor controllers |
| [ecmc](https://epics-modules.github.io/ecmc/) | EtherCAT motion controller and generic I/O controller support |
| [urRobot](https://epics-modules.github.io/urRobot/) | Support for Universal Robots e-series robot arms |
| [SyringePump](https://github.com/epics-modules/SyringePump) | Syringe pump control |

### Data Acquisition & Analysis

| Module | Description |
|--------|-------------|
| [mca](https://epics-modules.github.io/mca/) | Multi-channel analyzer (MCA) and multi-channel scaler (MCS) support |
| [dxp](https://epics-modules.github.io/dxp/) | Support for XIA digital x-ray spectroscopy electronics |
| [dxpSITORO](https://epics-modules.github.io/dxpSITORO/) | Driver for XIA SITORO-based FalconX spectrometers |
| [Dante](https://epics-modules.github.io/Dante/) | Support for Dante digital pulse processors |
| [ketek](https://epics-modules.github.io/ketek/) | Driver for KETEK silicon drift diode systems with DPP3 |
| [xspress3](https://epics-modules.github.io/xspress3/) | areaDetector driver for Xspress3 electronics |
| [sscan](https://epics-modules.github.io/sscan/) | Scanning support (step scans, fly scans) |
| [scaler](https://epics-modules.github.io/scaler/) | Scaler record support |

### Measurement & I/O Hardware

| Module | Description |
|--------|-------------|
| [measComp](https://epics-modules.github.io/measComp/) | Support for Measurement Computing USB and Ethernet I/O modules |
| [LabJack](https://epics-modules.github.io/LabJack/) | Support for LabJack I/O modules |
| [quadEM](https://epics-modules.github.io/quadEM/) | Support for quad electrometers and picoammeters |
| [ip330](https://epics-modules.github.io/ip330/) | Support for Acromag IP330 16-bit ADC Industry Pack modules |
| [ip230A](https://github.com/epics-modules/ip230A) | Support for Acromag IP230A 8-channel 16-bit DAC Industry Pack modules |
| [dac128V](https://epics-modules.github.io/dac128V/) | Support for Systran DAC-128V 8-channel 12-bit DAC Industry Pack modules |
| [ipUnidig](https://epics-modules.github.io/ipUnidig/) | Support for Industry Pack digital I/O modules |
| [Yokogawa_DAS](https://epics-modules.github.io/Yokogawa_DAS/) | Driver support for Yokogawa GM10 and MW100 data acquisition systems |
| [microEpsilon](https://github.com/epics-modules/microEpsilon) | Device support for Micro-Epsilon displacement measurement digitizers |
| [ThorLabsDFM](https://github.com/epics-modules/ThorLabsDFM) | Driver for ThorLabs deformable mirrors |

### Database & Record Support

| Module | Description |
|--------|-------------|
| [calc](https://epics-modules.github.io/calc/) | Calculation records (transform, sCalcout, aCalcout, and more) |
| [busy](https://epics-modules.github.io/busy/) | Busy record for ca_put_callback completion signaling |
| [std](https://epics-modules.github.io/std/) | Standard records and databases (EPID, timer, shutter, etc.) |
| [ip](https://epics-modules.github.io/ip/) | Industry Pack and serial/GPIB device databases |
| [delaygen](https://epics-modules.github.io/delaygen/) | Delay generator support |
| [love](https://epics-modules.github.io/love/) | Love controller support |
| [optics](https://epics-modules.github.io/optics/) | Optics devices (monochromators, tables, slits, filters, etc.) |
| [symb](https://github.com/epics-modules/symb) | Global symbol device support |

### IOC Infrastructure

| Module | Description |
|--------|-------------|
| [autosave](https://epics-modules.github.io/autosave/) | Automatic save and restore of PV values across IOC reboots |
| [iocStats](https://github.com/epics-modules/iocStats) | IOC status and resource monitoring |
| [alive](https://epics-modules.github.io/alive/) | IOC heartbeat and status reporting |
| [caPutLog](https://github.com/epics-modules/caPutLog) | Channel Access put logging (also supports PVA puts) |
| [caputRecorder](https://epics-modules.github.io/caputRecorder/) | Record and replay sequences of CA puts |
| [sequencer](https://epics-modules.github.io/sequencer/) | EPICS State Notation Language (SNL) sequencer |
| [pcas](https://github.com/epics-modules/pcas) | Portable Channel Access server library |

### Scripting & Automation

| Module | Description |
|--------|-------------|
| [lua](https://epics-modules.github.io/lua/) | Lua scripting for IOC shells, records, and device support |
| [pyDevSup](https://epics-modules.github.io/pyDevSup/) | EPICS device support in Python |

### VME, CAMAC & Bus Support

| Module | Description |
|--------|-------------|
| [vme](https://epics-modules.github.io/vme/) | VME bus support |
| [ipac](https://github.com/epics-modules/ipac) | IPAC carrier and communication module drivers |
| [camac](https://epics-modules.github.io/camac/) | CAMAC crate controller and module support |
| [devlib2](https://epics-modules.github.io/devlib2/) | Helper library for memory-mapped bus access |
| [SIS3153](https://github.com/epics-modules/SIS3153) | Driver for SIS3150/SIS3153 USB-to-VME interface cards |
| [vac](https://epics-modules.github.io/vac/) | Vacuum controller support |

### FPGA & Digital I/O

| Module | Description |
|--------|-------------|
| [softGlue](https://epics-modules.github.io/softGlue/) | FPGA-based configurable digital logic |
| [softGlueZynq](https://epics-modules.github.io/softGlueZynq/) | softGlue for Zynq-based platforms |
| [mrfioc2](https://epics-modules.github.io/mrfioc2/) | Driver for Micro Research Finland event timing system devices |

### Transient Recorders

| Module | Description |
|--------|-------------|
| [gtr](https://github.com/epics-modules/gtr) | Generic transient recorder support |
| [TRCore](https://github.com/epics-modules/TRCore) | General transient recorder framework |
| [TRGeneralStandards](https://github.com/epics-modules/TRGeneralStandards) | Transient recorder driver for General Standards 16AI64SSA/C |
| [TRSIS](https://github.com/epics-modules/TRSIS) | Transient recorder driver for SIS 3302 |
| [transRecorder](https://github.com/epics-modules/transRecorder) | Top-level directory for transient recorder software |

### Testing & Utilities

| Module | Description |
|--------|-------------|
| [gtest](https://github.com/epics-modules/gtest) | Google Test / Google Mock framework with EPICS integration |
| [MCoreUtils](https://epics-modules.github.io/MCoreUtils/) | Real-time utilities for EPICS IOCs on multi-core Linux |

### Templates & Examples

| Module | Description |
|--------|-------------|
| [xxx](https://epics-modules.github.io/xxx/) | synApps example/template IOC application |
| [ioczed](https://github.com/epics-modules/ioczed) | Template IOC for microzed/picozed platforms |

---

## Getting Started

Most modules follow a standard build process:

1. Clone the module repository:

   ```
   git clone https://github.com/epics-modules/<module>.git
   ```

2. Edit `configure/RELEASE` to set paths to EPICS base and any
   dependencies.

3. Build:

   ```
   make
   ```

For a complete synApps distribution that builds many of these modules
together, see the
[assemble_synApps](https://github.com/EPICS-synApps/assemble_synApps)
script.

## Contributing

Contributions are welcome. To contribute to an existing module:

1. Open an issue on the module's GitHub repository to discuss the change.
2. Fork the repository and make your changes on a branch.
3. Submit a pull request.

To add a new module to the organization, see
[Contributing a Module](contributing).

For questions and discussion, see the
[EPICS Tech-Talk mailing list](https://epics.anl.gov/tech-talk/) and the
[EPICS community](https://epics-controls.org/).

## Links

- [epics-modules on GitHub](https://github.com/epics-modules)
- [EPICS Controls](https://epics-controls.org/)
- [synApps](https://www.aps.anl.gov/BCDA/synApps)
- [EPICS Tech-Talk](https://epics.anl.gov/tech-talk/)
