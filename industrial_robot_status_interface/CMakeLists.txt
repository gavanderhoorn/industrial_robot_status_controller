cmake_minimum_required(VERSION 3.5)
project(industrial_robot_status_interface)

find_package(catkin REQUIRED
  COMPONENTS
    hardware_interface
)

catkin_package(
  INCLUDE_DIRS
    include
  CATKIN_DEPENDS
    hardware_interface
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
