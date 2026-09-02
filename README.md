# robot-description-urdf-xacro

A URDF and Xacro reference: the same mobile robot written twice, once in plain URDF and
once in Xacro, so the difference is visible side by side rather than described.

**Scope: this is a learning reference, not a robot model you should ship.** It is here
because a clear, honest comparison of the two formats — including the places where the
naive version breaks — is genuinely useful, and most examples online are either trivial
or enormous.

This repository merges two earlier repos (`mobile_robot_URDF` and
`Mobile_robot_xacro_n_xml`) that covered the same ground.

---

## What is in it

### `my_robot.urdf` — plain URDF

A differential-drive mobile robot: a box chassis, two drive wheels on continuous joints,
a spherical caster, and a cylindrical LiDAR on a fixed joint on top. Every link,
material and joint is written out longhand.

| Component | Shape | Size |
|---|---|---|
| Base | box | 0.6 x 0.4 x 0.2 |
| Wheels | cylinder | r 0.05, l 0.02 |
| LiDAR | cylinder | r 0.1, l 0.2 |
| Caster | sphere | r 0.05 |

This version is visual only — no `<inertial>` and no `<collision>` elements. That is
deliberate and it is the point of the comparison: it displays correctly in RViz and
behaves badly or not at all in Gazebo, because a physics engine needs mass and inertia.
A link with no inertial is one of the most common reasons a robot spawns and immediately
falls through the world or spins to infinity.

### `my_robot_using_xacro.urdf` — Xacro

The same structure, with:

- **Parameterised dimensions** — `base_width`, `base_length`, wheel radius and so on
  declared as properties, so changing the chassis size is one edit instead of six.
- **An inertia macro** — `<xacro:macro name="box_inertia">` computes the inertia tensor
  from mass and dimensions instead of leaving placeholder numbers. Cylinder and sphere
  equivalents follow the same pattern.
- **A more complete model** — base extensions, legs, head, arms, and in the URDF version
  a mesh-based gripper.

---

## Known problems in these files

Left as-is, and documented, because they are the failure modes worth recognising.

**Left and right components are still duplicated.** The macro handles inertia but not
symmetry. A properly factored description would define a wheel macro taking a name and a
reflect parameter (`+1` / `-1`) and instantiate it twice. Currently each side is written
out. This is the most common half-finished state a Xacro file ends up in.

**All the upper-body joints are fixed.** The robot has an arm and a head that cannot
move. Fine for visualisation, useless for control. Actuating them needs `revolute` or
`prismatic` joints with limits, and `<transmission>` or `<ros2_control>` blocks.

**The gripper meshes will not resolve.** The URDF version references
`package://urdf_tutorial/meshes/l_finger.dae` and similar. That package is not a
dependency here, so the mesh path fails and the link renders as nothing. `package://`
paths resolve against your workspace, not against the internet — a broken mesh path
usually shows up as a silently invisible link rather than an error, which is why it is
worth knowing about.

---

## Viewing it

```bash
git clone https://github.com/Pratyush150/robot-description-urdf-xacro.git
```

Plain URDF, straight into RViz:

```bash
ros2 run robot_state_publisher robot_state_publisher my_robot.urdf &
ros2 run joint_state_publisher_gui joint_state_publisher_gui &
rviz2
# Add a RobotModel display, set Fixed Frame to base_link
```

Xacro must be expanded first:

```bash
xacro my_robot_using_xacro.urdf > /tmp/robot.urdf
check_urdf /tmp/robot.urdf          # verifies the tree, prints the link hierarchy
ros2 run robot_state_publisher robot_state_publisher /tmp/robot.urdf
```

`check_urdf` is the fastest way to catch a second root link or an orphaned joint. Do it
before RViz, not after.

---

## File map

```
my_robot.urdf                  plain URDF: links, joints, materials, visual only
my_robot_using_xacro.urdf      Xacro: properties, inertia macro, fuller model
README.md
```

---

## What this is and is not

**It is** a compact reference for URDF structure, the URDF-to-Xacro transition, and
inertia macros, with the common mistakes left visible and labelled.

**It is not** a robot description you should base a real platform on. Dimensions are
illustrative, the kinematics are not actuated, and there are no Gazebo sensor or control
plugins.

For a description that is actually simulated and driven — Gazebo plugins, LiDAR and
camera sensors, a working diff-drive setup — see
[ros2-diffdrive-robot](https://github.com/Pratyush150/ros2-diffdrive-robot).

---

## Related work

Actively developed engineering tools:

| Repo | What it does |
|---|---|
| [px4-mavlink-companion](https://github.com/Pratyush150/px4-mavlink-companion) | MAVLink bridge, stale-telemetry watchdog, offboard control, serial auto-discovery |
| [flight-log-analyzer](https://github.com/Pratyush150/flight-log-analyzer) | PX4 ULog / ArduPilot log analysis producing a ranked findings report |
| [jetson-realtime-detection](https://github.com/Pratyush150/jetson-realtime-detection) | Real-time detection and tracking with per-stage latency profiling |
| [lidar-slam-toolkit](https://github.com/Pratyush150/lidar-slam-toolkit) | LiDAR SLAM configs plus extrinsics, time-sync and drift diagnostics |
| [drone-control-toolkit](https://github.com/Pratyush150/drone-control-toolkit) | PID with anti-windup, cascaded loops, LQR, EKF and complementary estimators |
| [ros2-drone-bringup](https://github.com/Pratyush150/ros2-drone-bringup) | ROS 2 bringup for a PX4 aircraft: geodesy, missions, geofence, SITL |

---

## License

MIT. Copyright (c) 2026 Pratyush Vatsa
