# Multi-host Bios update Support

Author: Priyatharshan P, [priyatharshanp@hcl.com]

Other contributors: None

Created: Nov 03,2020

## Problem Description

The current implementation in the phosphor-bmc-code-mgmt supports only single host
bios update.

As the open BMC architecture is evolving, the single host support becomes
contingent and needs multiple-host bios update to be implemented.

## Requirements


## Proposed Design

Create a property that the user specifies for which Host they want to update. 
This would require creating the /xyz/openbmc_project/software/hostX interfaces 
and the new property when the BMC starts up. Then for example:
 - User sets the new "this should be updated" property for host1 and
host3.
 - User calls the Redfish API and select the bios image to upload.
 - The BMC updater sees it's a host bios image and calls the bios_X updater 
script for only the host  instances that have the "this should be updated" 
property set.
 - The bios- image will be deleted after successfully updating all the hosts 
having "this should be updated" property set.
