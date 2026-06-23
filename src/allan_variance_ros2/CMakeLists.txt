cmake_minimum_required(VERSION 3.8)
project(allan_variance_ros2)

# ROS2 Rolling requiring C++17 minimum for modern headers like rosbag2_cpp
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Core Dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rosbag2_cpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(yaml-cpp REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem)
find_package(OpenMP REQUIRED)

# Fetch Content setup for external SDKs
include(FetchContent)
FetchContent_Declare(
  rerun_sdk
  URL https://github.com/rerun-io/rerun/releases/latest/download/rerun_cpp_sdk.zip
)
FetchContent_MakeAvailable(rerun_sdk)

# Include directories configuration
include_directories(include)

# 1. Main Shared Library Target
add_library(${PROJECT_NAME} SHARED
  src/ImuMeasurement.cpp
  src/yaml_parsers.cpp
  src/AllanVarianceComputor.cpp
)

# Explicitly link modern interface targets cleanly
target_link_libraries(${PROJECT_NAME} PUBLIC
  rclcpp::rclcpp
  rosbag2_cpp::rosbag2_cpp
  std_msgs::std_msgs
  sensor_msgs::sensor_msgs
  Eigen3::Eigen
  yaml-cpp
  Boost::filesystem
  OpenMP::OpenMP_CXX
)

# 2. Executable: allan_variance
add_executable(allan_variance src/allan_variance.cpp)
target_link_libraries(allan_variance PRIVATE ${PROJECT_NAME})

# 3. Executable: plot_imu
add_executable(plot_imu src/plot_imu.cpp)
target_link_libraries(plot_imu PRIVATE ${PROJECT_NAME} rerun_sdk)

# Install targets mappings
install(DIRECTORY include/ DESTINATION include)

install(TARGETS
  ${PROJECT_NAME}
  allan_variance
  plot_imu
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})

# Dynamic downstream exports
ament_export_dependencies(
  rclcpp 
  rosbag2_cpp 
  std_msgs 
  sensor_msgs 
  Eigen3 
  yaml-cpp 
  Boost
)
ament_package()