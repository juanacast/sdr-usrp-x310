# USRP X310 Hardware Overview

## 1. Introduction

The USRP X310 is a high-performance Software Defined Radio (SDR)
platform developed by Ettus Research. It is designed for wireless
communications, signal processing, prototyping, spectrum analysis,
and other applications requiring flexible radio-frequency hardware.

Unlike a conventional radio, where most signal-processing functions
are implemented by dedicated hardware, an SDR performs a significant
part of the processing digitally and allows its behavior to be
configured through software.

The X310 architecture is composed of three main processing stages:

1. RF frontend, implemented by interchangeable daughterboards.
2. High-speed ADC/DAC conversion and FPGA-based digital processing.
3. Communication with a host computer through high-speed interfaces.

The device is controlled from the host using the USRP Hardware Driver
(UHD), which provides access to parameters such as center frequency,
sample rate, RF gain, antenna selection, clock source, and timing
configuration.


## 2. USRP X310 Motherboard

The USRP X310 is the main processing platform of the SDR system.

Its architecture includes:

- Xilinx Kintex-7 XC7K410T FPGA
- Two RF daughterboard slots
- High-speed ADC and DAC converters
- 1 GB DDR3 memory
- Dual SFP+ interfaces
- PCI Express interface
- Built-in JTAG interface
- External reference clock and PPS interfaces
- Optional internal GPS Disciplined Oscillator (GPSDO)
- Digital GPIO interface

The motherboard itself does not define the final RF frequency range.
Instead, the RF operating range depends on the daughterboard installed
in the device.


## 3. FPGA

At the core of the USRP X310 is a Xilinx Kintex-7 XC7K410T FPGA.

The FPGA provides the high-speed digital processing and routing between
the RF frontends and the host interface.

With the standard UHD FPGA image, some of its main functions include:

- Digital Down Conversion (DDC)
- Digital Up Conversion (DUC)
- Digital frequency translation
- Interpolation and decimation
- Sample routing
- Timed commands
- Timed sampling
- Communication with the host computer
- Control of the RF daughterboards

The FPGA can also be reprogrammed to implement custom DSP architectures.

The X310 includes a built-in JTAG interface that allows direct access
to the FPGA. This interface is particularly useful for FPGA
development, debugging, and recovery when a valid FPGA image cannot
be loaded through the normal network interface.


## 4. RF Daughterboard — CBX

The RF frontend used with the system is the Ettus Research CBX
daughterboard.

The CBX is a wideband transceiver covering approximately:

    1.2 GHz – 6 GHz

It provides one transmit frontend and one receive frontend and supports
full-duplex operation.

The CBX uses independent local oscillators and synthesizers for its
transmit and receive paths. Therefore, the transmitter and receiver
can operate at different center frequencies.

Main characteristics:

| Parameter | CBX |
|---|---|
| Frequency range | 1.2 GHz – 6 GHz |
| Standard bandwidth | 40 MHz |
| TX gain range | 0 – 31.5 dB |
| RX gain range | 0 – 31.5 dB |
| TX antenna port | TX/RX |
| RX antenna ports | TX/RX or RX2 |
| RF impedance | 50 Ω |

> Note: CBX and CBX-120 are different variants. The standard CBX
> provides 40 MHz RF bandwidth, while CBX-120 provides 120 MHz.


## 5. RF Ports

The CBX daughterboard exposes two main RF connectors:

### TX/RX

The `TX/RX` connector is a shared RF port.

It can be used for:

- Signal transmission
- Signal reception

When configured for transmission, the generated RF signal is available
through this connector.

When no transmission is taking place, the same connector can be
selected as the receive input.

This allows a single antenna to be used for transmit and receive
applications where simultaneous transmission and reception are not
required.


### RX2

The `RX2` connector is a dedicated receive input.

Unlike the TX/RX connector, RX2 is not used for signal transmission.

It is particularly useful when separate transmitting and receiving
antennas are required.

For example:

    TX antenna  → TX/RX
    RX antenna  → RX2

This configuration is especially relevant for full-duplex operation.
When the CBX operates in full-duplex mode, reception is performed
through RX2.


## 6. Simplified RF Signal Paths

### Reception

A simplified receive chain can be represented as:

    Antenna
       │
       ▼
    CBX RF Frontend
       │
       ▼
    ADC
       │
       ▼
    FPGA
    (DDC / Decimation / DSP)
       │
       ▼
    Host Interface
       │
       ▼
    UHD
       │
       ▼
    GNU Radio / Python / C++

The incoming analog RF signal is first processed by the CBX
daughterboard.

The signal is then digitized by the X310 ADC and processed by the FPGA.
The resulting complex I/Q samples are transferred to the host computer,
where they can be processed using UHD-compatible software.


### Transmission

The transmission path follows the reverse direction:

    GNU Radio / Python / C++
       │
       ▼
    UHD
       │
       ▼
    Host Interface
       │
       ▼
    FPGA
    (Interpolation / DUC / DSP)
       │
       ▼
    DAC
       │
       ▼
    CBX RF Frontend
       │
       ▼
    TX/RX Port
       │
       ▼
    Antenna

Digital I/Q samples generated by the host are transferred to the FPGA,
processed and converted into an analog signal by the DAC.

The CBX then performs the required RF frequency conversion before the
signal is transmitted through the TX/RX port.


## 7. Host Communication Interfaces

The X310 provides multiple high-speed interfaces for communication
between the SDR and the host computer.

### SFP+ Interfaces

Two SFP+ ports are available and can be configured for Ethernet
communication.

Depending on the FPGA image and network configuration, these interfaces
can support Gigabit Ethernet or 10 Gigabit Ethernet operation.

The required interface bandwidth depends mainly on:

- Sample rate
- Number of channels
- Sample format
- Direction of communication (RX, TX or full duplex)

For basic operation, Ethernet provides a convenient interface between
the X310 and a host running UHD.


### PCI Express

The X310 also supports PCI Express connectivity.

PCIe provides a high-bandwidth, low-latency connection and can be useful
for applications requiring high sustained sample rates or deterministic
data transfer.


## 8. GPSDO and GPS Antenna

The X310 can be equipped with an internal GPS Disciplined Oscillator
(GPSDO).

A GPSDO combines a high-stability oscillator with timing information
obtained from the GPS constellation.

When a GPS antenna is connected and the GPSDO obtains a valid GPS lock,
the system can provide two important timing references:

- A high-accuracy 10 MHz frequency reference
- A 1 Pulse Per Second (1 PPS) timing reference

These references allow the SDR to maintain accurate frequency and time
synchronization.


### GPS ANT Port

The `GPS ANT` connector is used to connect the GPS antenna associated
with the internal GPSDO.

This connector is not an RF input for normal signal acquisition.

Its purpose is to receive signals from GPS satellites so that the GPSDO
can discipline the internal oscillator and establish an accurate timing
reference.


## 9. Clock and Time Synchronization

The X310 separates the concepts of frequency synchronization and time
synchronization.

### Frequency Reference

The frequency reference determines the accuracy of the clocks used for
signal generation, sampling, and RF synthesis.

Typical clock sources include:

- Internal oscillator
- External reference
- GPSDO

When the GPSDO is selected, the internal oscillator is disciplined using
GPS and provides a high-accuracy frequency reference.


### PPS Reference

The Pulse Per Second signal provides an accurate time boundary.

UHD can use the PPS event to align the internal device time with an
external timing reference.

This is important for applications involving:

- Timestamped samples
- Synchronized measurements
- Multiple SDR systems
- Time-aligned signal acquisition
- Radar and communication experiments


## 10. External Synchronization Ports

In addition to the internal GPSDO, the X310 provides external
synchronization connectors.

These include:

- `REF IN` — external reference clock input
- `REF OUT` — reference clock output
- `PPS IN` — external PPS timing input
- `PPS OUT` — PPS timing output

These interfaces allow the X310 to participate in larger synchronized
measurement systems without relying exclusively on its internal clock
or GPSDO.


## 11. JTAG Interface

The X310 includes a built-in USB-accessible JTAG interface.

JTAG provides direct access to the Kintex-7 FPGA and can be used with
Xilinx Vivado tools.

Typical uses include:

- FPGA development
- FPGA debugging
- Temporary FPGA image loading
- Device recovery

A bitstream loaded directly through JTAG configures the FPGA
temporarily. The configuration is lost after the device is powered
off unless the corresponding image is subsequently written to the
appropriate non-volatile storage.


## 12. Software Architecture

The main software interface for the USRP X310 is UHD
(USRP Hardware Driver).

A simplified software stack can be represented as:

    Application
    ├── GNU Radio
    ├── Python
    └── C++
          │
          ▼
         UHD
          │
          ▼
    Ethernet / PCIe
          │
          ▼
       USRP X310
          │
          ├── FPGA
          │
          ├── ADC / DAC
          │
          └── CBX RF Frontend

UHD abstracts most low-level hardware configuration and provides an API
for controlling parameters such as:

- Center frequency
- Sample rate
- RF gain
- Antenna selection
- Channel selection
- Clock source
- Time source
- Device timestamps
