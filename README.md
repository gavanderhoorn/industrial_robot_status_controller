# industrial_robot_status_controller


## Overview

These packages provide a `ros_control` compatible controller that publishes "robot status" of (industrial) robots in the form of [RobotStatus][] messages.

`hardware_interface`s need to expose an `IndustrialRobotStatusHandle` resource, which they update with information from a/the robot controller. The `IndustrialRobotStatusController` will then take this and populate the fields of the `RobotStatus` message with the information from the `IndustrialRobotStatusHandle`.

Note: the controller does not implement any logic to *derive* the values of the fields in `RobotStatus`, it merely transforms `IndustrialRobotStatusHandle` into a `RobotStatus` message. The `hardware_interface` is responsible for implementing the correct logic to populate the fields based on information about the status of the robot controller that it interfaces with.


## Usage

### Interface

`#include` the `industrial_robot_status_interface/industrial_robot_status_interface.h` header and store an instance of `industrial_robot_status_interface::IndustrialRobotStatusInterface` somewhere in the `hardware_interface`. Store an instance of `IndustrialRobotStatusHandle` as well.

Register the `IndustrialRobotStatusHandle` with the `IndustrialRobotStatusInterface` and finally register the `IndustrialRobotStatusInterface` with `ros_control` via `InterfaceManager::registerInterface(..)`.

Periodically update the `IndustrialRobotStatusHandle` (typically in `RobotHW::read(..)`) and set the fields (to `TriState::UNKNOWN`, `TriState::FALSE` or `TriState::TRUE`), based upon information from the robot controller available to the `hardware_interface`.

### Controller

As the controller does not implement any logic itself, but merely transforms a `IndustrialRobotStatusHandle` into a `RobotStatus` message, it's fully reusable and requires no source code changes (custom robot status derivation logic should be implemented in the `hardware_interface`).

The controller supports two configurable settings:

 - `publish_rate`: rate at which `RobotStatus` messages should be published (default: 10 Hz)
 - `handle_name` : name of the `IndustrialRobotStatusHandle` resource exposed by the `hardware_interface` (default: `"industrial_robot_status_handle"`)

See the *Example* section below for a full configuration example of the controller.


## Example


### Interface

Initialising and registering the `IndustrialRobotStatusHandle` and `IndustrialRobotStatusInterface`:

```c++
...

// these could be class members of course
industrial_robot_status_interface::RobotStatus robot_status_resource_{};
industrial_robot_status_interface::IndustrialRobotStatusInterface robot_status_interface_{};

...

// somewhere in the initialisation of the hardware_interface
robot_status_interface_.registerHandle(
  industrial_robot_status_interface::IndustrialRobotStatusHandle(
    "industrial_robot_status_handle", robot_status_resource_));
registerInterface(&robot_status_interface_);

...

```

Note: the first argument to the `IndustrialRobotStatusHandle` ctor can be anything, as long as the controller is configured to look for the same resource handle name. The default is `"industrial_robot_status_handle"`.

Finally: update the members of `robot_status_resource_` with data from the controller:

```c++
...

// somewhere in RobotHW::read(..)
using industrial_robot_status_interface::TriState;
using industrial_robot_status_interface::RobotMode;
using fanuc::stream_motion::bit_on;

// set defaults
robot_status.in_motion       = TriState::UNKNOWN;
// skip other fields for brevity
robot_status.mode            = RobotMode::UNKNOWN;
robot_status.error_code      = 0;

// use controller info to set real values
if (latest_robot_data_.state == MyRobot::IS_MOVING)
  robot_status.in_motion = TriState::TRUE;

...

```

### Controller

Add the controller as usual to a `ros_control` configuration `.yaml`:

```yaml
robot_status_controller:
  type: industrial_robot_status_controller/IndustrialRobotStatusController
  handle_name: my_custom_status_handle
  publish_rate: 10

```

Here the `IndustrialRobotStatusHandle` was registered under a different resource name, so `handle_name` is set to `"my_custom_status_handle"`.


## Additional information

Refer to the [RobotStatus][] message documentation for more information on how the values for the fields should be derived.


[RobotStatus]: http://docs.ros.org/latest/api/industrial_msgs/html/msg/RobotStatus.html
