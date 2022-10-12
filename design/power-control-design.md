# phosphor-state-manager power-control enhancement

Author:
   Karthikeyan P(Karthik), [p_karthikeya@hcl.com](mailto:p_karthikeya@hcl.com)
   Naveen Moses S(naveen.moses), [naveen.mosess@hcl.com](mailto:naveen.mosess@hcl.com)
   Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)

other contributors:

created:
    Oct 11, 2022

## Problem Description

In our machine, the complete power control will be done using the 
phosphor-state-manager

In phosphor-state-manager makes extensive use of systemd. It
follows the use of systemd targets, so we preferred to control
our system uses the phosphor-state-manager because in our system 
the GPIOs are not used (or not available) directly to control the
power of the system. if GPIOs are used we thought of going with 
x86-power-control.

** supported transition items in the phosphor-state-manager

The following states and states requests are already available in
phosphor-state-manager those are

   * Host: RequestedHostTransition: On, Off, Reboot.

   * Chassis: RequestedPowerTransition: On, Off.

** Not supported transition items

1. In our system there is a separate power line under the sub-chassis (slot 
   power) for that, there is a support, target, or state map which is not 
   available, so needs to introduce a new DBUS object in the phosphor-
   state-manager to handle the slot power(sub-chassis).
   
   In x86-power-control the above mentioned requirements are handled
   with chassis_system1..N for slot power on, off, and reboot. So
   need to implement the targets for the sub-chassis (chassis_system) 
   to control the power of the slot in the phosphor-state-manager.

   for slot(sub-chassis) power on, off, and reboot, The following state 
   request needs to be implemented for sub-chassis in the 
   phosphor-state-manager.

    * sub-chassis power on (slot power on)
    * sub-chassis power off (slot power off)
    * sub-chassis power reboot (slot power reboot)

2. In the existing phosphor-state manager for chassis the following 
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

The below-mentioned DBUS object is created to handle the slot power on,
off, and reboot in x86-power-control. So the same thing needs to
implement here as well. The reference link is mentioned here.

   * Interface: xyz.openbmc_project.State.Chassis
   * Object path: /xyz/openbmc_project/state/chassis_system1
                  /xyz/openbmc_project/state/chassis_system2
                  .
                  .
                  .
                  /xyz/openbmc_project/state/chassis_systemN


link for the reference:
https://github.com/openbmc/x86-power-control/blob/master/src/power_control.cpp#L3058

## Requirements

1. The slot power on, off, and reboot state transition request needs to implement
   in the phosphor-state-manager. For example, the transition request 
   implementation for the slot on, off, and reboot should be,

    * Interface: xyz.openbmc_project.State.Chassis
    * object path: /xyz/openbmc_project/state/chassis_system1
                  .
                  .
                  .
                  /xyz/openbmc_project/state/chassis_systemN
    * Transition: On, Off, Reboot

2. For chassis Reboot transition request is required for the chassis
   power cycle it should be,

    * Chassis: RequestedPowerTransition: Reboot

3. Add support for checking pgood status of the multi-host system in
   the phosphor-state-manager.

4. Add support for state transition request to sled for complete sled
   cycle. and also need introduce a new object chassis_system0 for
   the sled cycle.

    * Object path: /xyz/openbmc_project/state/chassis_system0
    * RequestedPowerTransition: Reboot

## Proposed Design

1. This document proposes a design to implement the following state request
   transition for slot power on, off, and reboot in the phosphor-state-manager.

   The implementation for slot power off/on/reboot transition should be

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
   The following interface is created for slot power on, off, and reboot
   state transition request

   ```
   xyz.openbmc_project.State.Chassis

   ```

   The following property is the part of slot power on and off state
   transition request

   ```
   .RequestedPowerTransition - This property shows the power transition
                        (xyz.openbmc_project.State.Chassis.Transition.On/Off/Reboot)

   ```
   Based on the transition request the respective mapped target will call
   and bash script will take the remaining action, based on transition request.

2. For chassis need to implement power cycle or reboot transition 
   request in the phosphor-state-manager. It should be

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
   -manager only supports single(single-host) property read for pgood status,
   So need to change or add support to read (or monitor) the pgood status for
   multi host system.

4. For the sled power cycle need to implement or introduce new object and
   transition state, it should be

   Example :

   ```
     /xyz/openbmc_project/state/chassis_system0
     .RequestedPowerTransition - Reboot

   ```
## Impact

The change only adds the new DBUS interface to discover the slot power
transition state request so there is no impact in case it is the impact
another platform we can implement only in our system.

 Example : set flag for only in our system.

## Testing

The change tested with greatlakes platform.
