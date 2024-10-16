# UWB Gazebo Sensor Plugin

<p align="center">
    <a href="https://github.com/AUVSL/UWB-Gazebo-Plugin/graphs/contributors" alt="Contributors">
        <img src="https://img.shields.io/github/contributors/AUVSL/UWB-Gazebo-Plugin" /></a>
    <a href="https://github.com/AUVSL/UWB-Gazebo-Plugin/pulse" alt="Activity">
        <img alt="GitHub commit activity" src="https://img.shields.io/github/commit-activity/m/AUVSL/UWB-Gazebo-Plugin"></a>
    <a href="https://github.com/AUVSL/UWB-Gazebo-Plugin/stargazers">
        <img alt="Stars" src="https://img.shields.io/github/stars/AUVSL/UWB-Gazebo-Plugin"></a>
    <a href="https://github.com/AUVSL/UWB-Gazebo-Plugin/network/members">
        <img alt="Forks" src="https://img.shields.io/github/forks/AUVSL/UWB-Gazebo-Plugin"></a>
    <a href="https://github.com/AUVSL/UWB-Gazebo-Plugin/issues">
        <img alt="Issues" src="https://img.shields.io/github/issues/AUVSL/UWB-Gazebo-Plugin"></a>
    <a href="./LICENSE" alt="Activity">
        <img alt="GitHub" src="https://img.shields.io/github/license/AUVSL/UWB-Gazebo-Plugin"></a>
</p>

**NOTE:** This repository is related with the next scientific work and was forked from this [repository](https://github.com/valentinbarral/gazebosensorplugins):

**Barral, V.**; **Escudero, C.J.**; **García-Naya, J.A.**; **Maneiro-Catoira, R.** *NLOS Identification and Mitigation Using Low-Cost UWB Devices.* Sensors 2019, 19, 3464.[https://doi.org/10.3390/s19163464](https://doi.org/10.3390/s19163464)

If you use this code for your scientific activities, a citation is appreciated.

## README

This project contains several plugins to use in Gazebo simulator:

### Requirements

Libraries ```libignition-math4-dev``` and ```libgazebo9-dev``` must be installed before building this package.

Also, package ```gtec_msgs``` must be present in the same work space. This package can be found here:  [https://github.com/AUVSL/rosmsgs](https://github.com/AUVSL/rosmsgs)

### Build

After cloning the repository in a catkin workspace:
```
$ catkin_make
```

### UWB Plugin

![GTEC UWB Plugin in RVIZ](https://user-images.githubusercontent.com/38099967/64428790-e66b6780-d0b4-11e9-8f6f-489d8eb949c8.png)

This plugin simulates a UWB tag sensor. The plugin simulates the reception of UWB ranging measurements from a set of anchors placed on the scenario. The plugin also produces different measurements depending on the type of line of sight between the tag and each anchor. Thus, four models are considered:

- LOS (Line of Sight): In this case there are no obstacles between the tag and the anchor. In this case, the ranging estimation is near to the actual value.
- NLOS Soft (Non Line of Sight Soft) : In this case, there is a thin obstacle between the tag and the anchor. The received power is lower than in the LOS case but the ranging estimation is closer to the actual value.
- NLOS Hard: In this case, there are too many obstacles between the tag and the anchor that the direct ray is unable to reach the receiver. But in this case, there is a path from the tag to the anchor after the signal rebounding in one wall. Thus, the estimated ranging is the corresponding to this new path, so is going to be always longer than the actual distance between the devices.
- NLOS: Tag and anchor are too far apart or there are too many obstacles between them, they are unable to comunicate and generate a ranging estimation.

NOTE: the rebounds are only computed using obstacles placed at the same height than the tag. That means that the rebounds on the floor or the ceil are not considered.

To add the plugin to a Gazebo model, the next code must be present in the .sdf o .urdf.

```xml
<plugin name='libgtec_uwb_plugin' filename='libgtec_uwb_plugin.so'>
  <update_rate>25</update_rate>
  <nlosSoftWallWidth>0.25</nlosSoftWallWidth>
  <tag_z_offset>1.5</tag_z_offset>
  <tag_link>tag_0</tag_link>
  <anchor_prefix>uwb_anchor</anchor_prefix>
  <all_los>false</all_los>
  <tag_id>0</tag_id>
  <return_angle>true</return_angle>
</plugin>
``` 

* update_rate: num. of rates published by second.
* nlosSoftWallWidth: how thin a wall must be to let the signal go through it. 
* tag_z_offset: a offset in meters to add to the current height of the sensor.
* anchor_prefix: all the anchors placed in the scenario must have a unique name that starts with this prefix.
* all_los: if true, all the anchors are considered as in a LOS scenario. Except if there are too far from the tag, in that case they are considered as NLOS.
* tag_id: tag identifier, a number.


To place an anchor in a Gazebo's world, the only requirement is that the model must have a name starting with ```anchor_prefix```. A simple model could be:

```xml
<model name="uwb_anchor0">
  <pose>0 0 1.5 0 0 0</pose>
  <static>1</static>
  <link name="link">
    <visual name="visual">
      <geometry>
        <box>
          <size>0.2 0.2 0.2</size>
        </box>
      </geometry>
    </visual>
  </link>
</model>
```

otherwise you can place an anchor in the world by naming a link with ```anchor_prefix```. This link can be defined in a SDF or a URDF as long as Gazebo can see it. For fixed joints you might need to enabled:

```xml
<link name="uwb_anchor0">
  <inertial>
    <mass value="1" />
    <origin rpy="0 0 0" xyz="0 0 0"/>
    <inertia ixx="1.0" ixy="0.0" ixz="0.0" iyy="1.0" iyz="0.0" izz="1.0" />
  </inertial>

  <collision>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <box size="1 1 1"/>
    </geometry>
  </collision>
  <visual>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <box size="1 1 1"/>
    </geometry>
  </visual>
</link>

<joint name="uwb_anchor_joint" type="fixed">
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <parent link="base_link"/>
  <child link="uwb_anchor0"/>
</joint>

<gazebo reference="uwb_anchor_joint">
  <preserveFixedJoint>true</preserveFixedJoint>
</gazebo>

<gazebo reference="uwb_anchor0">
  <material>Gazebo/White</material>
</gazebo>
```

This plugin publish the next topics:

- ```/gtec/gazebo/uwb/ranging/tag_id``` : where ```tag_id``` is the value configured in the plugins. This topic publish the UWB range estimations using messages of type: gtec_msgs/Ranging.msg ([https://github.com/valentinbarral/rosmsgs](https://github.com/valentinbarral/rosmsgs))

- ```/gtec/gazebo/uwb/anchors/tag_id``` : where ```tag_id``` is the value configured in the plugins. This topic publish the position of the UWB anchors in the scenario. Each anchor have a different color depending of its current LOS mode: green-LOS, yellow-NLOS Soft, blue-NLOS Hard and red-NLOS. The published messages are of type visualization_msgs/MarkerArray.msg ([visualization_msgs/MarkerArray Message](http://docs.ros.org/melodic/api/visualization_msgs/html/msg/MarkerArray.html))



