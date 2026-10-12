# unitree_hg

ROS 2 message definitions for Unitree's "HG" robots (G1, H1-2): `LowCmd`, `LowState`, `MotorCmd`,
`MotorState`, `IMUState`, hand, BMS and pressure sensor messages.

The fields match the DDS types in Unitree's firmware. A ROS 2 node using CycloneDDS on domain 0
can therefore read the robot's topics (such as `rt/lowstate`) and publish commands (such as
`rt/lowcmd` or `rt/arm_sdk`) directly, without Unitree's SDK. Commands also need Unitree's CRC
over this exact layout; see `motor_crc_hg.cpp` in unitree_ros2's examples.

## Why this package exists on its own

It lets you write ordinary ROS 2 packages for Unitree robots that depend only on the message
definitions. You build it with `colcon` next to your own packages and list it in their
`package.xml`, as with any interface package.

Unitree's [unitree_ros2](https://github.com/unitreerobotics/unitree_ros2) also contains these
messages, but as part of a self-contained setup with its own assumptions. With a stock ROS 2
install (e.g. Jazzy from apt), most of that setup isn't needed:

- **A source build of CycloneDDS and `rmw_cyclonedds`.** The robot runs CycloneDDS 0.10.x. Unitree
  has you build that version from source because ROS 2 Foxy ships an older, incompatible one.
  Humble and later ship a 0.10.x CycloneDDS with their binary `rmw_cyclonedds_cpp`, which talks to
  the robot as is. A separate DDS build only adds an underlay to keep in sync.
- **Setup scripts.** `setup.sh` and its variants source `/opt/ros/foxy`, expect the repo at
  `$HOME/unitree_ros2`, and hard-code the network interface in an inline `CYCLONEDDS_URI`. With a
  normal install, you set `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp`, `ROS_DOMAIN_ID=0` and a
  `CYCLONEDDS_URI` file for your own interface, the same way as in any other ROS 2 project (shell,
  launch file or container).
- **RTI Connext IDL generation.** Unitree's CMake requires `rosidl_generator_dds_idl` to also emit
  Connext IDL. That generator isn't part of a standard install, and CycloneDDS doesn't use its
  output. This copy makes it optional (see below).
- **Packages for other robots and APIs.** `unitree_go` (the Go2, B2 and H1 messages, plus shared
  types such as `SportModeState` and `WirelessController`) and `unitree_api` (request/response
  types for Unitree's high-level services) come along whether or not you need them. If you need
  one, copy it the same way.
- **Examples.** The `example` package is useful for reference, such as the CRC code and the motor
  index tables, but it isn't a library to depend on.

## Origin and changes

This is a copy of `cyclonedds_ws/src/unitree/unitree_hg` from unitree_ros2 at commit `3ff13ea`
(BSD 3-Clause, © Unitree Robotics).

The message files are unchanged. The only change is in `CMakeLists.txt`: the Connext IDL step is
optional (`find_package(rosidl_generator_dds_idl QUIET)` and an `if` around
`rosidl_generate_dds_interfaces`). If the generator is installed, it still runs.

To update, copy `msg/` from a newer unitree_ros2 and keep the CMake change. Packages that compute
Unitree's CRC over these messages must be checked whenever the message layout changes.
