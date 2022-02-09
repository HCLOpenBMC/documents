# OCP Multi-host Platform specific OEM command Implementation

Author:
   Kumar Thangavel(kumar), [thangavel.k@hcl.com](mailto:thangavel.k@hcl.com)
   Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)

other contributors:

created:
    Feb 7, 2022

## Problem Description

The current implementation in openBMC supports BMC to host communication for
single and muli hosts. For BMC to host communication some OCP specific
platforms using Bridge-IC. This Bridge-IC acts as a bridge between BMC to host.
All the ipmb commands and OEM commands are passing through this Bridge-IC.

The current implementation in openBMC does not support OCP platform specific
oem command handling. openBMC does not support oem command for firmware update
of CPLD, VR, BIOS, BIC etc.

This design documents focuses on handling oem commands for firmware update of
CPLD, VR, BIOS, BIC, etc for OCP specific platforms. It involves new daemon and
repository creation for this implementation. This new repository name would be
BICcode.

## Background and References

The BMC to Host communication is happening via ipmbbridge and ipmid
daemons. These daemnos handling all the ipmb commands request and response for
BMC to host communication. These ipmb and OCP specific oem commands are sending
and receiving via Bridge-IC.

This Bridge-IC transfers all the ipmb commands from host to ipmmbridge daemon.
Some of the ipmb commands are get device id, get fru and sdr details, etc.
It forwards the request and response for multihost ipmb commands also.
This forwards OEM command request and response for firmware update of multiple
devices like CPLD, VR, BIC, BIOS etc for multi host platforms.

This Bridge-IC has some intelligence. It has the specification. Based on the
spec, It will receives the request and adding some BIC headers like BIC netfn,
BIC cmd and IANA etc as prefix with that oem commands and send those commands
to particular handlers functions. Based on the netfn and cmd that handler
function got invoked and parsed the BIC headers and data part and processed
the commands send back the response to BIC. BIC remove the BIC headers and
forward the response to the requester.

## Requirements

* BIC code should handles all the oem commands requests like firmware update of
  CPLD, VR, Bios etc.
* BIC code should process all the oem commands.
* BIC code should send the response back to the oem command requester.
* New repository should be created for handling these BIC codes.
* Support added for single and multi-host oem firmware update.

## Proposed Design

This document proposes a new design engaging the OEM commands request and
response for firmware update of CPLD, VR, BIOS, etc, and it's implementation
details.

The implementation flow of OEM firmware update:

1) Seperate daemon will be created for this OEM formware update.
2) Binary file and slotId details can be an input from the user.
3) Image can be generated from binary file and it has been validated.
4) If image is not valid, then will stop this flashing and firmware update.
5) If image is valid, then get the device details from the user. The device
which needs to updated. Ex CPLD, VR, BIOS, etc.
6) Target can be set for firmware update. Target will be CPLD, VR, BIOS, etc
7) ME recovery mode has been set
8) The OEM command will be framed as Ipmb command with netfn and cmd and sent
via Bridge-IC and send to the device which needs to be do the firmware update
9) If any error response, then write flashing and update is considered as
failed.
10) If no errors, then flashing and firmware update is success.

## Alternatives Considered

1) fb-ipmi-oem - An approach has been tried with fb-ipmi-oem repository. This
   is library code and part of ipmid daemon. This libraray code handles platform
   specific and oem commands request and response. But this code, cannot do
   firmware update. OEM commands for firmware update cannot initiated from
   fb-ipmi-oem. Because fb-ipmi-oem code handles only incoming oem commands
   request using handler functions and send the response back to the requester
   command.

2) openbmc-tools - An another approach has been tried with openbmc-tools
   repository. This openbmc-tools repository is mainly used for having debug
   and build tools and scripts. This implementation is specific to OCP
   platforms and does not having C++ code. Also, MAINTAINERS not suggested to
   use this repository.

## Impacts

This is only for OCP platform specific implementation and these codes lies in
new repository. So, there is no impact for others repositories and other
platforms.

## Testing

Testing this OEM commands like firmware update for CPLD, VR and Bios with yosemitev2
platform.

