# Copyright(c) 2020 livoxtech limited.

cmake_minimum_required(VERSION 3.14)

include(cmake/version.cmake)
project(
  livox_ros_driver2
  VERSION ${LIVOX_ROS_DRIVER2_VERSION}
  LANGUAGES C CXX
)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()

message(STATUS "${PROJECT_NAME} version: ${PROJECT_VERSION}")

find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(std_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)

# Generate the Livox custom ROS messages.
set(LIVOX_INTERFACES livox_interfaces2)

rosidl_generate_interfaces(${LIVOX_INTERFACES}
  "msg/CustomPoint.msg"
  "msg/CustomMsg.msg"
  DEPENDENCIES
    builtin_interfaces
    std_msgs
  LIBRARY_NAME ${PROJECT_NAME}
)

# Locate the installed Livox-SDK2 library and headers.
find_library(
  LIVOX_LIDAR_SDK_LIBRARY
  NAMES livox_lidar_sdk_shared
  HINTS /usr/local/lib /usr/local/lib64
  REQUIRED
)

find_path(
  LIVOX_LIDAR_SDK_INCLUDE_DIR
  NAMES
    livox_lidar_api.h
    livox_lidar_def.h
  HINTS /usr/local/include
  REQUIRED
)

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/livox_ros_driver2.cpp
  src/lddc.cpp
  src/driver_node.cpp
  src/lds.cpp
  src/lds_lidar.cpp

  src/comm/comm.cpp
  src/comm/ldq.cpp
  src/comm/semaphore.cpp
  src/comm/lidar_imu_data_queue.cpp
  src/comm/cache_index.cpp
  src/comm/pub_handler.cpp

  src/parse_cfg_file/parse_cfg_file.cpp
  src/parse_cfg_file/parse_livox_lidar_cfg.cpp

  src/call_back/lidar_common_callback.cpp
  src/call_back/livox_lidar_callback.cpp
)

target_compile_definitions(${PROJECT_NAME}
  PRIVATE BUILDING_ROS2
)

target_include_directories(${PROJECT_NAME}
  PRIVATE
    ${PCL_INCLUDE_DIRS}
    ${LIVOX_LIDAR_SDK_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_link_directories(${PROJECT_NAME}
  PRIVATE
    ${PCL_LIBRARY_DIRS}
)

target_compile_options(${PROJECT_NAME}
  PRIVATE
    ${PCL_DEFINITIONS}
)

# Link generated CustomMsg and CustomPoint C++ type support.
rosidl_get_typesupport_target(
  cpp_typesupport_target
  ${LIVOX_INTERFACES}
  rosidl_typesupport_cpp
)

target_link_libraries(${PROJECT_NAME}
  ${LIVOX_LIDAR_SDK_LIBRARY}
  ${PCL_LIBRARIES}
  Eigen3::Eigen
  ${cpp_typesupport_target}
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "livox_ros::DriverNode"
  EXECUTABLE ${PROJECT_NAME}_node
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(
  INSTALL_TO_SHARE
    config
    launch
)