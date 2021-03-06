# Copyright (C) 2008-2013 LAAS-CNRS, JRL AIST-CNRS.
#
# Author: Florent Lamiraux, Nirmal Giftsun
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

# Catkin part

cmake_minimum_required(VERSION 2.4.6)

project(dynamic_graph_bridge)

find_package(catkin REQUIRED COMPONENTS roscpp rospy std_msgs message_generation std_srvs geometry_msgs sensor_msgs tf realtime_tools dynamic_graph_bridge_msgs)
find_package(Boost REQUIRED COMPONENTS program_options)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ros_interpreter ros_bridge
  CATKIN_DEPENDS roscpp message_runtime)

include_directories(${catkin_INCLUDE_DIRS})
include_directories(${PROJECT_SOURCE_DIR}/include)

set(CXX_DISABLE_WERROR False)
set(CUSTOM_HEADER_DIR dynamic_graph_bridge)
set(${PROJECT_NAME}_HEADERS
  include/dynamic_graph_bridge/ros_init.hh
  include/dynamic_graph_bridge/ros_interpreter.hh
  )
INSTALL(FILES ${${PROJECT_NAME}_HEADERS}
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
include(cmake/base.cmake)
include(cmake/ros.cmake)
include(cmake/GNUInstallDirs.cmake)
include(cmake/python.cmake)

set(EXECUTABLE_OUTPUT_PATH ${CATKIN_DEVEL_PREFIX}/bin)
set(LIBRARY_OUTPUT_PATH ${CATKIN_DEVEL_PREFIX}/lib)
set(CMAKE_INSTALL_RPATH "${LIBRARY_OUTPUT_PATH}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${LIBRARY_OUTPUT_PATH}")
set(PKG_CONFIG_ADDITIONAL_VARIABLES
  ${PKG_CONFIG_ADDITIONAL_VARIABLES}
  plugindirname
  plugindir
  )

add_required_dependency(bullet)
add_required_dependency(jrl-mal)
add_required_dependency(dynamic-graph)
add_required_dependency(dynamic-graph-python)
add_required_dependency(sot-core)
add_required_dependency(sot-dynamic)
add_required_dependency(jrl-dynamics-urdf)

add_library(ros_bridge
  src/converter.hh
  include/dynamic_graph_bridge/ros_init.hh src/ros_init.cpp
  src/sot_to_ros.hh src/sot_to_ros.cpp
  )
pkg_config_use_dependency(ros_bridge jrl-mal)
pkg_config_use_dependency(ros_bridge bullet)
target_link_libraries(ros_bridge ${catkin_LIBRARIES})
install(TARGETS ros_bridge DESTINATION lib)

# Make sure rpath are preserved during the install as ROS dependencies
# are not installed.
set_target_properties(ros_bridge PROPERTIES BUILD_WITH_INSTALL_RPATH True)

macro(compile_plugin NAME)
  message(lib path ${LIBRARY_OUTPUT_PATH})
  file(MAKE_DIRECTORY "${LIBRARY_OUTPUT_PATH}/dynamic_graph/ros/${NAME}")
  add_library(${NAME} src/${NAME}.cpp src/${NAME}.hh)
  pkg_config_use_dependency(${NAME} jrl-mal)
  pkg_config_use_dependency(${NAME} dynamic-graph)
  pkg_config_use_dependency(${NAME} sot-core)
  pkg_config_use_dependency(${NAME} jrl-dynamics-urdf)
  target_link_libraries(${NAME} ${catkin_LIBRARIES})
  add_dependencies(${NAME} ros_bridge)
  target_link_libraries(${NAME} ros_bridge)
  set_target_properties(${NAME} PROPERTIES BUILD_WITH_INSTALL_RPATH True)
  install(TARGETS ${NAME} DESTINATION lib)

  dynamic_graph_python_module("ros/${NAME}"
    ${NAME}
    ros/${NAME}/wrap
    )

  PKG_CONFIG_USE_DEPENDENCY(ros/${NAME}/wrap jrl-mal)
  PKG_CONFIG_USE_DEPENDENCY(ros/${NAME}/wrap dynamic_graph)
  PKG_CONFIG_USE_DEPENDENCY(ros/${NAME}/wrap sot-core)
  target_link_libraries(ros/${NAME}/wrap ${catkin_LIBRARIES})
  file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}/dynamic_graph/ros/${NAME})
  set_target_properties(ros/${NAME}/wrap PROPERTIES
                        LIBRARY_OUTPUT_DIRECTORY ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION})
  file(COPY ${PROJECT_BINARY_DIR}/src/dynamic_graph/ros/${NAME}/__init__.py
       DESTINATION ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_PYTHON_DESTINATION}/dynamic_graph/ros/${NAME})
endmacro()

include(cmake/python.cmake)

compile_plugin(ros_publish)
compile_plugin(ros_subscribe)
compile_plugin(ros_time)
compile_plugin(ros_joint_state)

target_link_libraries(ros_joint_state "${DYNAMIC_GRAPH_PLUGINDIR}/dynamic.so")

compile_plugin(robot_model)

# ros_interperter library.
add_library(ros_interpreter src/ros_interpreter.cpp)
pkg_config_use_dependency(ros_interpreter jrl-mal)
pkg_config_use_dependency(ros_interpreter dynamic-graph)
pkg_config_use_dependency(ros_interpreter sot-core)
target_link_libraries(ros_interpreter ${catkin_LIBRARIES})

add_dependencies(ros_interpreter ros_bridge)
target_link_libraries(ros_interpreter ros_bridge)
set_target_properties(ros_interpreter PROPERTIES BUILD_WITH_INSTALL_RPATH True)
message(cmakeinstalllibdir ${CMAKE_INSTALL_LIBDIR} )
install(TARGETS ros_interpreter DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

# Stand alone remote dynamic-graph Python interpreter.
add_executable(interpreter src/interpreter.cpp)
add_dependencies(interpreter ros_interpreter)
target_link_libraries(interpreter ros_interpreter)
pkg_config_use_dependency(interpreter jrl-mal)
pkg_config_use_dependency(interpreter dynamic-graph)
pkg_config_use_dependency(interpreter sot-core)
pkg_config_use_dependency(interpreter sot-dynamic)
target_link_libraries(interpreter ${catkin_LIBRARIES})
# set_target_properties(interpreter PROPERTIES BUILD_WITH_INSTALL_RPATH True)
#install(TARGETS interpreter DESTINATION bin)

# Stand alone embedded intepreter with a robot controller.
add_executable(geometric_simu src/geometric_simu.cpp src/sot_loader.cpp)
target_link_libraries(geometric_simu ${catkin_LIBRARIES} ${Boost_LIBRARIES} dl)

add_subdirectory(src)

#install ros executables
install(PROGRAMS
  ${PROJECT_SOURCE_DIR}/scripts/robot_pose_publisher
  ${PROJECT_SOURCE_DIR}/scripts/run_command
  ${PROJECT_SOURCE_DIR}/scripts/tf_publisher
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
