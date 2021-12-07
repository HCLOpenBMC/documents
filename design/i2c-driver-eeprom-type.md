# EEPROM device type identification based on platform specific 

Author:
   Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)
   Karthikeyan P(karthik), [pkarthikey@hcl.com](mailto:pkarthikey@hcl.com)

other contributors:

created:
    Dec 3, 2021

## Problem Description
Current implementation of EEPROM one byte or two byte identification
not working properly for our platform, Also not able to fix this issue
without EEPROM write. It may corrupt EEPROM. So, we cannot
use the existing logic.

We have an FRU that contain types of oem, one type of oem supports
one byte EEPROM another type of oem supports two byte EEPROM,
so we need to identify which type of EEPROM byte type is present
for our platform.

## Background and References
In our platform we have a two type of NIC cards, that is Broadcom NIC
and Mellanox NIC, Broadcom NIC supports one byte EEPROM and Mellanox
NIC supports two byte EEPROM.

This can be identified using platform specific service file(shell script)
using NIC manufacturer infrormation.

Here, we mentioned link of NIC specification,
https://www.opencompute.org/documents/facebook-ocp-mezzanine-20-specification

## Requirements
This feature is needed to support or update the value at run time, then
entity-manager configuration will update or populate in the dbus as a
property, based on this we can confirm that which type of byte in NIC card
used in particular yosemitev2 either Broadcom or Mellonox.

## Proposed Design
In this implementation creates a new platform specific cpp(.cpp)file in
entity-manager to read the script file, and which is used to identify
the byte type to update the EEPROM type dynamically at the run time
as well as dbus property.

## Alternatives Considered 
We tried with an approch to identify the EEPROM device byte
type, for this created a platform specific service(.service) file to
parse the script output to the machine layer and try to read the byte
type using FBYV2.json under probe of decorator.compatible
mentioned below,
"xyz.openbmc_project.Inventory.Decorator.Compatible"
it should be readable not writeable. if it could be writeable in
decorator.compatible it should be easy to update as a dbus property.
but we couldn't able to set the value in dbus dynamically at run time
using set property command.

## Impacts
This is for machine specific so less impact.

## Testing
Testing with yosemitev2 platform.
