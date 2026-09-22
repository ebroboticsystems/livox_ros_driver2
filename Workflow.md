# How it works

1. Startup and configuration \\

The entry point in livox_ros_driver2.cpp starts the ROS node and reads parameters like:
xfer_format
publish_freq
frame_id
user_config_path

2. LiDAR SDK initialization \\

In lds_lidar.cpp, the driver:
parses the LiDAR config file,
initializes the Livox SDK,
registers the LiDAR device,
sets up the callback handlers.


3. Raw packet reception \\


The callback layer in pub_handler.cpp receives raw Ethernet packets from the sensor.
It separates them into:
point cloud packets
IMU packets \\

The same file converts raw point data into structured points with:

position (x,y,z)
(x,y,z)
intensity
line number
tag
timestamp
It also applies extrinsic calibration so the points are transformed into the correct coordinate frame.

4. Publishing to ROS \\ 

The distribution layer in lddc.cpp packages the processed data into ROS messages and publishes them on topics such as:
livox/lidar
livox/imu