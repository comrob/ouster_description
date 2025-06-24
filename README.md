# Ouster OS1 Sensor Description and Visualization

A minimal ROS package to visualize an Ouster OS1-64 sensor model in RViz against an existing, namespaced TF tree.

## Overview

This package provides a simple way to display the 3D model of an Ouster OS1-64 sensor in RViz. It is specifically designed for situations where another ROS node or a rosbag is already publishing the sensor's coordinate frames (TFs), especially when those frames are published with a ROS namespace (e.g., `/os1/os_sensor`).

It correctly handles the common problem of linking a URDF model (which uses a `tf_prefix` with underscores, like `os1_lidar`) to an existing TF tree that uses namespaces (like `os1/os_lidar`).

## Prerequisites

* ROS Noetic (or a similar version with `catkin`).
* A running `roscore`.
* An active data source (e.g., a `rosbag play` command or a live driver) that is publishing the sensor's TF frames.

## Installation

1.  Clone this repository into your catkin workspace's `src` directory:
    ```bash
    cd ~/your_catkin_ws/src/
    git clone <URL_to_your_repository>
    ```

2.  Build the workspace:
    ```bash
    cd ~/your_catkin_ws/
    catkin_make
    ```

3.  Source the workspace to make the package available in your environment:
    ```bash
    source devel/setup.bash
    ```

## Usage

This package is designed to run alongside your existing data source. You will typically use two terminals.

#### Terminal 1: Your Data Source

In your first terminal, start the node or rosbag that publishes your sensor's data and TF frames.

**Example using `rosbag play`:**
```bash
# This bag must publish TF frames like /os1/os_sensor, /os1/os_lidar, etc.
rosbag play your_ouster_data.bag --clock
```

#### Terminal 2: Launch the Visualization

In your second terminal, run the launch file from this package.
```bash
roslaunch ouster_description visualize.launch tf_prefix:=os1
```

### The `tf_prefix` Argument

The `tf_prefix` argument is crucial. **It must match the namespace used in your TF data.**

* If your frames are `os1/os_sensor`, `os1/os_lidar` -> use `tf_prefix:=os1`
* If your frames are `my_robot/sensor_link`, `my_robot/lidar_link` -> use `tf_prefix:=my_robot`

You should now see an RViz window with the Ouster sensor model correctly positioned relative to your other TF frames.

## How It Works

This package uses a few key components to solve the visualization problem:

1.  **`urdf/my_ouster_model.urdf.xacro`**: This is the top-level robot description file. It defines a dummy `world` link and instantiates the Ouster model from `OS1-64.urdf.xacro`, attaching it to this `world` link.

2.  **`launch/visualize.launch`**: This is the main entry point. It does three things:
    * Loads the URDF from `my_ouster_model.urdf.xacro` to the `/robot_description` parameter.
    * Starts the `robot_state_publisher` to publish the internal transforms of the sensor model (e.g., the transform from `os1` to `os1_imu`).
    * **Starts a `static_transform_publisher` to create a "bridge".**

### The TF Bridge

The key to this package is the `static_transform_publisher` node launched in `visualize.launch`. It publishes a single, static transform that connects the namespaced frame from your data source (e.g., `os1/os_sensor`) to the `world` link defined in our URDF.

This bridge is what solves the "No transform from \[world\] to \[base_footprint\]" error in RViz, snapping the sensor model into the correct place within your robot's main TF tree.
