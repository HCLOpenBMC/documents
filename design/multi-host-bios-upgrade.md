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
 - The property "this should be updated" will be set to default after the successful
completion of bios-update.

## Alternatives Considered

### Design 1

 - Number of host will be identified from machine layer [OBMC_HOST_INSTANCES]

 - Code will be modified to create n number of objects based on number of hosts

  Ex: Log taken in YosemiteV2 [4 host]

root@yosemitev2:~# busctl tree xyz.openbmc_project.Software.BMC.Updater
`-/xyz
  `-/xyz/openbmc_project
    `-/xyz/openbmc_project/software
      |-/xyz/openbmc_project/software/1929c585
      |-/xyz/openbmc_project/software/host1
      | `-/xyz/openbmc_project/software/host1/28bd62d9
      |-/xyz/openbmc_project/software/host2
      | `-/xyz/openbmc_project/software/host2/28bd62d9
      |-/xyz/openbmc_project/software/host3
      | `-/xyz/openbmc_project/software/host3/28bd62d9
      `-/xyz/openbmc_project/software/host4
        `-/xyz/openbmc_project/software/host4/28bd62d9
root@yosemitev2:~# busctl tree xyz.openbmc_project.Software.Version
`-/xyz
  `-/xyz/openbmc_project
    `-/xyz/openbmc_project/software
      |-/xyz/openbmc_project/software/host1
      | `-/xyz/openbmc_project/software/host1/28bd62d9
      |-/xyz/openbmc_project/software/host2
      | `-/xyz/openbmc_project/software/host2/28bd62d9
      |-/xyz/openbmc_project/software/host3
      | `-/xyz/openbmc_project/software/host3/28bd62d9
      `-/xyz/openbmc_project/software/host4
        `-/xyz/openbmc_project/software/host4/28bd62d9


 - This will create activation interface for each host. For a multi-host system if the  RequestedActivation is set to "xyz.openbmc_project.Software.Activation.RequestedActivations.Active", then different bios service file will be called based the host.

For single host : biosServiceFile = "obmc-flash-host-bios@" + versionId + ".service";
For multi host  : biosServiceFile =  "obmc-flash-host" + host + "-bios@" + versionId + ".service";


Then it can be used for multi host even if the firmware image we want to install is the same for multiple host targets.

#### Reason for rejection

Even after the successful activation of bios update, the image will not be deleted from the BMC.We dont want to potentially fill up the /tmp space.

### Design 2

Host id will be added as an array of host id in extra version field in the 
MANIFEST file.Format can be defined, this way as soon as we see bios image 
uploaded and based on extra version, it will create as many version interface 
and allow to activate them. Once we call all the bios upgrade script, we should 
delete image.

#### Reason for rejection

The MANIFEST file is an  factory ship-out, it may not be appropriate to add the 
host IDs into it.


