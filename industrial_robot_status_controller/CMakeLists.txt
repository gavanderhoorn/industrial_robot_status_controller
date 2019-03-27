cmake_minimum_required(VERSION 3.5)
project(industrial_robot_status_controller)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

find_package(catkin REQUIRED
  COMPONENTS
    controller_interface
    hardware_interface
    industrial_msgs
    industrial_robot_status_interface
    pluginlib
    realtime_tools
)

catkin_package(
  LIBRARIES
    ${PROJECT_NAME}
  CATKIN_DEPENDS
    controller_interface
    hardware_interface
    industrial_msgs
    industrial_robot_status_interface
    pluginlib
    realtime_tools
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS})

add_library(${PROJECT_NAME} src/controller.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})

install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})

install(FILES plugin.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
