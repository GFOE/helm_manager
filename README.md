# Helm Manager

The helm_manager is a Project11 package that helps manage the flow of control commands. It responds to changes in piloting mode to determine which helm or command velocity source to direct to the underlying node controlling the hardware. It also sends a heartbeat message indicating the current piloting mode and status information from the helm node.

Functionally seems to be analogous to the [twist_mux](https://wiki.ros.org/twist_mux) node.

Acts as a mode-dependent mux (input selection) for either helm or twiststamped messages. And enforces the max_speed and max_yaw_speed limits set by ROS parameters.

## Piloting modes

It appears that the user can specify arrays/lists of "piloting modes".  There are two lists of modes: "active" modes and "standby" modes.  The "active" modes have "output enabled".  I think "output enabled" actually means that the node then subscribes to the "helm" and "cmd_vel" topics for that mode.

The default setup is to have:

* active modes:  ['manual', 'autonomous']
* standby modes: ['standby']

The actual mode of operation is the internal state of the node that determines which subscribed topic (twiststamped/helm) is then output.  (I'll call this the AMO becuase the use of active/enabled/standby is very confusing in this context.) The AMO is set by the subscription to `piloting_mode`.  There can only be one AOM at any time.  All the piloting modes publish on their own `active` topics to indicate if they are the current AMO. (Based on the source it appears that there is no validation on the String received on the `piloting_mode` subscription, so not sure what happens if there is an error in the String.)

## Control message types and conversion

The publish/subscribe types and behavior is cotnrolled by the `~output_type` parameter which leads to something akin to ROS polymorphism.  The two output types are "helm" and "twist" (more apt name would be "twiststamped")

Helm and TwistStamped

## Node: helm_manager

### Subscribed Topics

* `piloting_mode` (std_msgs::String)
* `status/helm` (marine_msgs::Heartbeat)

#### Subscribers for active piloting modes
For each piloting mode in the "active" list has "output enabled" which it publishes to `helm` and `cmd_vel` topics.

* `MODE_PREFIX/MODE/helm`(marine_msgs::Helm)
* `MODE_PREFIX/MODE/cmd_vel`(geometry_msgs::TwistStamped)

### Published Topics

* `Heartbeat` (marine_msgs::Heartbeat)

#### If `~output_type` param is "twist"

* `out/cmd_vel`(geometry_msgs::TwistStamped).

#### If `~output_type` param is "helm"

* `out/helm`([marine_msgs::Helm](https://github.com/GFOE/marine_msgs/blob/gfoe-devel/msg/Helm.msg)).

#### Publications for piloting modes
For all piloting modes---both the "active" and "standby" modes, which can have "output enabled" as either true of false--the node publishes to an 'active' topic:

* `MODE_PREFIX/MODE/active`(std_msgs::Bool)

### Parameters

* `~output_type` (String): Default `helm`.  Can be `helm` or `twist`
* `~piloting_mode_prefix` (String)
* `piloting_modes/active` (Array of Strings)
* `piloting_modes/standby` (Array of Strings)

#### If `~output_type` param is "twist"

* `~twist/max_speed`(Double): Default 1.0
* `~twist/max_yaw_speed` (Double): Default 1.0

#### If `~output_type` param is "helm"

* `~helm/max_speed`(Double)
* `~helm/max_yaw_speed` (Double)

### Examples

#### Annie Example

[launch/fy21_step2_annie_common.launch](https://bitbucket.org/gfoe/project12/src/gfoe-devel/launch/fy21_step2_annie_common.launch), which uses [config/annie_helm_manager.yaml](https://bitbucket.org/gfoe/project12/src/gfoe-devel/config/annie_helm_manager.yaml)

Pertinent bits from the launch file:
```
<remap from="out/cmd_vel" to="control/cmd_vel"/>
```
 
Pertinent configuration from yaml file includes:
```
helm_manager/output_type: 'twist'
helm_manager/piloting_modes/active: ['manual', 'autonomous']
helm_manager/piloting_modes/standby: ['standby']
helm_manager/piloting_mode_prefix: 'piloting_mode/'
```

The node is launched within the `annie` namespace.

Resulting Subscribers:

* `annie/piloting_mode/manual/cmd_vel`(geometry_msgs::TwistStamped)
* `annie/piloting_mode/manual/helm`([marine_msgs::Helm](https://github.com/GFOE/marine_msgs/blob/gfoe-devel/msg/Helm.msg))

Resulting Publishers

* `annie/project11/piloting_mode`(std_msgs::String)
* `annie/control/cmd_vel`(geometry_msgs::TwistStamped)
* `annie/piloting_mode/standby/active`(std_msgs::Bool)
* `annie/piloting_mode/manual/active`(std_msgs::Bool)
* `annie/piloting_mode/autonomous/active`(std_msgs::Bool)


