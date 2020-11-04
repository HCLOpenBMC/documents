
# Multi-host Bios update Support

Author: Priyatharshan P, [priyatharshanp@hcl.com]

Other contributors: None

Created: Nov 04,2020

## Problem Description

The current implementation in the phosphor-bmc-code-mgmt supports only single host bios update.

As the open BMC architecture is evolving, the single host support becomes contingent and needs multiple-host bios update to be implemented.

## Requirements
The multi-host bios update implementation considers the following requirement.
-  Single bios image can be used for multiple hosts bios update. 
-  User to configure the required hosts (multi-host) for bios update.
## Proposed Design
This document proposes a new design engaging a phosphor-bmc-code-mgmt to update bios image for multi host system. Provided below the implementation steps.
- phosphor-dbus-interfaces changes
	- Create a new biosUpdate interface
- phosphor-bmc-code-mgmt changes
	- Host Identification
	- New hostX objects creation
	- Bios image clean up
	- Property reset in biosUpdate interface
	- Multi-host bios update flow
- Redfish & busctl API

### Create a new biosUpdate interface:
In single host system the firmware update command can be called directly. But in multi host system before firmware update we need to specify some where what and all the hosts are going to be updated.

So the new interface with a property, where user specifies for which Host they want to update will be created in phosphor-dbus-interfaces.

The new interface will be having the following property.
```
properties:
    - name: biosUpdate
      type: boolean
      default: false
      description: >
        The host identifier for bios update.
```
### Host Identification
Number of hosts will be identified from machine layer (OBMC_HOST_INSTANCES)
N = OBMC_HOST_INSTANCES

### New hostX objects creation

create the /xyz/openbmc_project/software/hostX (Where X = 1,2,...N)  
with the new interface created.

For Example, In a multi host system [ 4 hosts] the objects will look like below:
```
root@yosemitev2:~# busctl tree xyz.openbmc_project.Software.BMC.Updater
`-/xyz
  `-/xyz/openbmc_project
    `-/xyz/openbmc_project/software
      |-/xyz/openbmc_project/software/a81bb606
      |-/xyz/openbmc_project/software/host1
      |-/xyz/openbmc_project/software/host2
      |-/xyz/openbmc_project/software/host3
      `-/xyz/openbmc_project/software/host4
 ```  

Before firmware update the user should specify what are all the hosts are going to be updated by setting the property biosUpdate to "true" in each host interfaces.

### Bios image clean up
In single host system the bios- image will be deleted after the successful firmware update.But here in multi-host system the image will be deleted only after successful updation of all hosts having  "biosUpdate" property set.

### Property reset in biosUpdate interface
After successful completion, the property "biosUpdate" will  set back to default value in all the hostX objects. So that it can be used for next bios update.

### Multi-host bios update flow
If the user wanted to update the same image for host1 and host 3 then:
 - User sets the new "biosUpdate" property for host1 and
host3.
 - User calls the Redfish API and select the bios image to upload.
 - The BMC updater sees it's a host bios image and calls the bios_X updater script for only the host  instances that have the "biosUpdate" property set.
 - The bios- image will be deleted after successfully updating all the hosts 
having "biosUpdate" property set.
 - The property "biosUpdate" will be set to default after the successful
completion of bios-update.

### Redfish & busctl API
In mullti-host system, before the User calls bios-update API,  he may need to call N number of set  API commands to set "biosUpdate" property per host.

For example, If the user wanted to update the same image for host1 and host 3 then the command would be look something like below.
For Host1:
busctl:
```
busctl set-property xyz.openbmc_project.Software.BMC.Updater /xyz/openbmc_project/software/host1 xyz.openbmc_project.Software.biosUpdate biosUpdate b true
```
Redfish:
```
curl -g -k -H "X-Auth-Token: $token" https://${bmc}/redfish/v1/Host/Actions/host1.TobeUpdated -d '{"Value": "true"}'
```
For Host3:
busctl:
```
busctl set-property xyz.openbmc_project.Software.BMC.Updater /xyz/openbmc_project/software/host3 xyz.openbmc_project.Software.biosUpdate biosUpdate b true
```
Redfish:
```
curl -g -k -H "X-Auth-Token: $token" https://${bmc}/redfish/v1/Host/Actions/host2.TobeUpdated -d '{"Value": "true"}'
```
Then to update the firmware,
busctl:
```
busctl set-property xyz.openbmc_project.Software.BMC.Updater /xyz/openbmc_project/software/<id> xyz.openbmc_project.Software.Activation RequestedActivation s xyz.openbmc_project.Software.Activation.RequestedActivations.Active
```
Redfish:
```
curl -u root:0penBmc -curl k -s  -H "Content-Type: application/octet-stream" -X POST -T <image file path> https://${BMC}/redfish/v1/UpdateService
```

## Alternatives Considered

### Design 1

1. Number of host will be identified from machine layer [OBMC_HOST_INSTANCES]

2. Code will be modified to create n number of objects based on number of hosts 
   like below

```
      |-/xyz/openbmc_project/software/host1
      | `-/xyz/openbmc_project/software/host1/28bd62d9
      |-/xyz/openbmc_project/software/host2
      | `-/xyz/openbmc_project/software/host2/28bd62d9
      |-/xyz/openbmc_project/software/host3
      | `-/xyz/openbmc_project/software/host3/28bd62d9
      `-/xyz/openbmc_project/software/host4
        `-/xyz/openbmc_project/software/host4/28bd62d9
```

3. This will create activation interface for each host. For a multi-host system 
if the  RequestedActivation is set to 
"xyz.openbmc_project.Software.Activation.RequestedActivations.Active", 
then different bios service file will be called based the host.

For single host : 
biosServiceFile = "obmc-flash-host-bios@" + versionId + ".service";
For multi host  : 
biosServiceFile =  "obmc-flash-host" + host + "-bios@" + versionId + ".service";

Then it can be used for multi host even if the firmware image we want to 
install is the same for multiple host targets.

#### Reason for rejection

Even after the successful activation of bios update, the image will not be deleted 
from the BMC.We dont want to potentially fill up the /tmp space.

### Design 2

Host id will be added as an array of host id in extra version field in the 
MANIFEST file.Format can be defined, this way as soon as we see bios image 
uploaded and based on extra version, it will create as many version interface 
and allow to activate them. Once we call all the bios upgrade script, we should 
delete image.

#### Reason for rejection

The MANIFEST file is an  factory ship-out, it may not be appropriate to add the 
host IDs into it.

## Impacts
None

## Testing
meta-yosemitev2 is a multi-host system [4 host]. This feature will be tested 
here in meta-yosemitev2.

