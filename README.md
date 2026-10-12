# unitree_hg

ROS 2 message definitions for Unitree's "HG" robots (G1, H1-2): `LowCmd`, `LowState`, `MotorCmd`,
`MotorState`, `IMUState`, hand, BMS and pressure sensor messages.

The fields match the DDS types in Unitree's firmware. That lets a plain ROS 2 node using CycloneDDS
on domain 0 read the robot's topics (`rt/lowstate`) and send commands (`rt/arm_sdk`, `rt/lowcmd`)
without Unitree's SDK.

## Who uses it

`unitree_g1_driver` includes `unitree_hg/msg/low_cmd.hpp` and `low_state.hpp`. Its CRC
(`motor_crc_hg.hpp`) is computed over this message layout, so the two must stay in sync with
Unitree's.

## Origin

This is a copy of `cyclonedds_ws/src/unitree/unitree_hg` from
[unitree_ros2](https://github.com/unitreerobotics/unitree_ros2) at commit `3ff13ea`
(BSD 3-Clause, © Unitree Robotics). It was copied so the workspace builds without the rest of
unitree_ros2, which is a full workspace with its own CycloneDDS build.

The message files are unchanged. The only change is in `CMakeLists.txt`: Unitree also generates
RTI Connext IDL (`rosidl_generator_dds_idl`). That generator isn't part of a standard Jazzy install
and isn't needed with CycloneDDS, so it is now optional (`find_package(... QUIET)` and an `if`).

To update, copy the `msg/` folder from a newer unitree_ros2 and keep the CMake change.
