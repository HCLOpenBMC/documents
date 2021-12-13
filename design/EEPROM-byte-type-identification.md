# Platform specific EEPROM device type identification

Author:
   Karthikeyan P(karthik), [pkarthikey@hcl.com](mailto:pkarthikey@hcl.com)
   Kumar Thangavel(kumar), [thangavel.k@hcl.com](mailto:thangavel.k@hcl.com)
   Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)

other contributors:

created:
    Dec 8, 2021

## Problem Description

The current implementation in entity-manager FRU device application
supports EEPROM single byte or two byte identification. But, this
implementation does not work in all the machine. Also there is a chance
of data corruption in the logic when it does EEPROM write.

We have an FRU that contain types of oem, one type of oem supports
one byte EEPROM another type of oem supports two byte EEPROM,
so we need to identify which type of EEPROM byte type is present
in yosemitev2 machine.

## Background and References

Existing FRU device implementation of device byte type identification:

The current entity-manager FRU device application identifies single byte or
two byte EEPROM type using byte read and write logic.

https://github.com/openbmc/entity-manager/blob/master/src/FruDevice.cpp#L197

Our yosemitev2 machine supports two types of NIC card, Broadcom NIC has
FRU details in single byte EEPROM and Mellanox NIC has FRU details
stored in two byte EEPROM.

Below is the link to OCP NIC specification,
https://www.opencompute.org/documents/facebook-ocp-mezzanine-20-specification
http://files.opencompute.org/oc/public.php?service=files&t=3c8f57684f959c5b7abe2eb3ee0705b4

## Requirements

* Need to identify single byte or two byte EEPROM (Broadcom or Mellanox).
* To get the EEPROM type single byte or two byte at platform level.
* Entity-manager should read this EEPROM type from machine layer.
* This byte type identification logic in entity-manager should be generic
  for all the platforms.

## Proposed Design

This document proposes a new design engaging the FRU device to read the
EEPROM device byte, single byte or two byte from the machine layer.

This implementation is proposed to initiate the service from entity-manager
configuration file, the service is initiate when the flag is set(machine specific)
and which identifies whether this particular machine has single byte EEPROM or
two byte EEPROM. then this EEPROM device type will be updated in the dbus property
by entity-manager.

In machine layer, EEPROM device byte type(single byte or two byte) can be
identified using platform specific service from NIC manufacturer
information. This service will be specific and installed in machine layer.

The implementation flow of EEPROM byte type identification:

1) In entity-manager json configuration file, creates a flag to enabling the
   service file, which is initiate when the flag is set.

2) Whenever the flag is set in entity-manager configuration, the particular machine
   specific code will be be executed otherwise the existing logic executes.

3) The sevice or script output(single byte or two byte) was given back to
   the service, where it's initiated.

4) Then the service or script output populates the EEPROM byte type in
   dbus property by entity-manager.

5) Finally the FRU device application will read from the dbus property and update
   which type of EEPROM either single byte EEPROM or two byte EEPROM in our
   yosemitev2 machine.

This byte type identification logic in entity-manager should be generic
for all the platforms.

Following modules will be updated for this implementation
* Entity-manager
* Machine layer

## Alternatives Considered

An approach has been tried with identifying the EEPROM byte type from the
platform specific service and added probe field with compatible string
in entity-manager platform specific config file. But, In entity-manager
dbus properties doesn't have write permission. So EEPROM byte type cannot
be updated in dbus property.

This new generic function creation approach in entity-manager can be more
suitable to handle EEPROM device byte type identification.

## Impacts

This is for machine specific so less impact.

## Testing

Testing with yosemitev2 platform.

