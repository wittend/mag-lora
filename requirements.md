# Requirements

## Goals

 A project for creating a remote sensor for gathering data from the RM-3100CM sensor module via a LoRa RF signalling.

### The deliverable items for this project are:
* A prototype design for a remotely operating data collection and transfer device that supports the PNI-RM3100CM module for field deployment.
* Firmware to make this device interoperable with the HAMSci Personal Space Weather Station and similar projects.
* Host-side hardware for the reception of the transmitted data.
* Host-side firmware for decoding data And interface to the HamSCI mag-usb software.
* A cost-conscious, production ready design for both host-side and remote-side hardware.

## Constraints

* Personal project, maintained by one person. 
* Prefer boring, standard tooling.
* C++ firmware.
* ESP32-S3 platform.
* Platformio Library Framework
* Compatibility with LoRA signalling.
* National and international emissions compliance.

## Non-goals

* Best effort delivery.
* No authentication.

## Notes

- (Open questions, ideas parked for later, links to relevant docs.)
