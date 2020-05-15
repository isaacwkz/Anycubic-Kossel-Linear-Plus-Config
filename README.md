# Anycubic Kossel Linear Plus Marlin 2.0 Config

## Hardware config
- BigTreeTech SKR Pro
- TMC2209 with sensorless homing
> Motors are powered from 24V while extruder and bed heaters are at 12V
- Stock Z-sensor
- Stock front panel

## Sensorless homing
- Homing current can be reduced to increase sensitivity when homing.

## Motor grinding when homing
Delta homing procedure happens in 2 stages:
  1. Fast move to bring end effctor up until one of the three carriage is at Z-min
  2. Individually home each of the three carriages.

This movement causes the first carriage that has already reach the top in the first pass to grind.

The motor grinds as sensorless homing does not work reliably when motor is not moving (since carriage is already at z-min)

## Fix
- Move all carriages down after the first pass to provide space for sensorless homing to trigger reliably.

Delta homing procedure is found in `src\Module\Delta.cpp` `void home_delta()`

The following block of code moves all 3 carriages up.
```
// Move all carriages together linearly until an endstop is hit.
current_position.z = (delta_height + 10
  ...
);
line_to_current_position(homing_feedrate(Z_AXIS));
planner.synchronize();
...
endstops.validate_homing_move();
```
Once the above set of blocking code has finished executing, one of the cariages has reached the top

Insert the following set of code to execute a blocking downwards move.
```
current_position.z = (delta_height -5); //commands a downwards z movement
line_to_current_position(homing_feedrate(Z_AXIS)); //executes queued movement
planner.synchronize(); //waits for queued movements to finish executing
```
The above set of code commands the cariages to move 5mm downwards, which is enough to trigger sensorless homing (on my machine hehe)

The rest of the homing procedure will resume as per usual, homing each carriage individually.
