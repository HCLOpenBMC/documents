# phosphor-state-manager power-control enhancement

Author:
   Karthikeyan P(Karthik), [p_karthikeya@hcl.com](mailto:p_karthikeya@hcl.com)
   Naveen Moses S(naveen.moses), [naveen.mosess@hcl.com](mailto:naveen.mosess@hcl.com)
   Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)

other contributors:

created:
    Oct 11, 2022

## Problem Description

The plan is to use phosphor-state-manager for the complete power-control of the
machine.

The platform or the machine which is targeted does not have the regular design
of having different GPIO lines for the power control operation.  The
x86-power-control is specifically designed to use the power control using GPIOs
and phosphor-state-manager look more suitable for this application as it uses a
systemd target and it could machine-specific operations. The power of the host
shall be controlled using ipmb interface or I2C with a set of oem specific ipmb
commands or I2C commands.

** supported transition items in the phosphor-state-manager

The following states and states requests are already available in
phosphor-state-manager those are

   * Host: RequestedHostTransition: On, Off, Reboot.

   * Chassis: RequestedPowerTransition: On, Off.

** Not supported transition items

1. In our system there is a separate power line under the sub-chassis (slot 
   power) for that, there is a support, target, or state map which is not 
   available, so needs to introduce a new DBUS object in the phosphor-
   state manager to handle the slot power(sub-chassis).
   
   In x86-power-control the above-mentioned requirements are handled
   with chassis_system1..N for slot power on, off, and reboot. So
   need to implement the targets for the sub-chassis (chassis_system) 
   to control the power of the slot in the phosphor-state-manager.

   for slot(sub-chassis) power on, off, and reboot, The following state 
   request needs to be implemented for sub-chassis in the 
   phosphor-state-manager.

    * sub-chassis power on (slot power on)
    * sub-chassis power off (slot power off)
    * sub-chassis power reboot (slot power cycle)

2. In the existing phosphor-state manager for chassis, the following
   supports are available that is  On and Off (RequestedPowerTransition).
   but for reboot transition request needs to implement in the
   phosphor-state-manager

   Example :
           Chassis: RequestedPowerTransition: Reboot

3. concerning multi-host systems the phosphor-state-manager
   considered as a single host and it's checked for single property for
   pgood, So need to change or add support to check pgood status for
   multi hosts system.

4. in our system one more additional power line which represents a sled
   power reset, For that need to add an object in phosphor-state-manager
   it should be

   Example :
           Object path: /xyz/openbmc_project/state/chassis_system0

## Background and References

The following are the dbus interfaces proposed to be created to handle the
slot power control, ie on, off, and, cycle.

   ```
   Interface: xyz.openbmc_project.State.Chassis

   Object path: /xyz/openbmc_project/state/chassis_system1
                  /xyz/openbmc_project/state/chassis_system2
                  .
                  .
                  .
                  /xyz/openbmc_project/state/chassis_systemN
   ```

link for the reference:
https://github.com/openbmc/x86-power-control/blob/master/src/power_control.cpp#L3058

## Requirements

1. The slot power control or sub-chassis power control is currently not
   supported and it is proposed to be added to the phosphor-state-manager.
   For example, the transition request implementation for the slot on, off,
   and slot power cycle should be,

    * Interface: xyz.openbmc_project.State.Chassis
    * object path: /xyz/openbmc_project/state/chassis_system1
                  .
                  .
                  .
                  /xyz/openbmc_project/state/chassis_systemN
    * Transition: On, Off, PowerCycle

2. Currently the chassis on/off is supported and it is proposed to add
   reboot along with on/off, it should be,

    * Chassis: RequestedPowerTransition: Reboot

3. Add support for monitoring power good status for the individual host and
   the sub-chassis. As per the current implementation, the power good
   status is monitored only for the power supply.

4. Add support for sled cycle, to control complete sled power cycle. The
   implementation for sled power cycle should be

    Example :

    ```
    Object path: /xyz/openbmc_project/state/chassis_system0
    RequestedPowerTransition: PowerCycle
    ```

## Proposed Design

1. This document proposes a design to implement the following state request
   transition for slot power on, off, and reboot in the phosphor-state-manager.

   The implementation for slot power off/on/cycle transition should be

   Example :

   ```
   `-/xyz
     `-/xyz/openbmc_project
       `-/xyz/openbmc_project/state
         `-/xyz/openbmc_project/state/chassis_system1

     `-/xyz/openbmc_project
       `-/xyz/openbmc_project/state
         `-/xyz/openbmc_project/state/chassis_system2
          .
          .
          .
   `-/xyz
     `-/xyz/openbmc_project
       `-/xyz/openbmc_project/state
         `-/xyz/openbmc_project/state/chassis_systemN
   ```
   The following interface is created for slot power on, off, and cycle state
   transition request

   ```
   xyz.openbmc_project.State.Chassis
   ```

   The following property is the part of slot power on and off state transition
   request

   ```
   .RequestedPowerTransition - This property shows the power transition
                        (xyz.openbmc_project.State.Chassis.Transition.On/Off/Powercycle)
   ```
   Based on the transition request the respective mapped target will call and
   the bash script will take the remaining action, based on the transition request.

2. For chassis need to implement a power cycle or reboot transition request in the
   phosphor-state-manager. It should be

   Example :

   ```
   `-/xyz
     `-/xyz/openbmc_project
       `-/xyz/openbmc_project/state
         `-/xyz/openbmc_project/state/chassis1

   .RequestedPowerTransition - This property shows the power transition
                          (xyz.openbmc_project.State.Chassis.Transition.Reboot)
   ```

3. To read pgood status of the multi-host system, in existing phosphor-state
   -manager only supports single(single-host) property read for pgood status, So
   need to change or add support to read (or monitor) the pgood status for multi
   host system.

   Each sub-chaasis, host, and slot power need initial power status, So
   need to determine the initial state of power. A separate process or application
   handle those initial status of power which has a property that holds the
   initial power state. Then the phosphor-state-manager will update or determine
   the initial state of power from it.

4. For the sled power cycle need to implement or introduce new object and
   transition state, and it should be

   Example :

   ```
     /xyz/openbmc_project/state/chassis_system0
     .RequestedPowerTransition - Reboot
   ```
## Impact

The change only adds the new DBUS interface to discover the slot power
transition state request so there is no impact if it's impact to another
the platform we can implement only in our system.

 Example: set flag for only in our system.

## Testing

The change tested with greatlakes platform.
