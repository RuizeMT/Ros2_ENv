cmake_minimum_required(VERSION 3.5)
project(ros2_env)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(gazebo_ros REQUIRED)
find_package(gazebo_dev REQUIRED)
find_package(gazebo_plugins REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

include_directories(include 
 	${gazebo_dev_INCLUDE_DIRS}
	${gazebo_ros_INCLUDE_DIRS}
  ${PROJECT_SOURCE_DIR}/include
  ${Boost_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${GAZEBO_INCLUDE_DIRS}
  ${GAZEBO_MSG_INCLUDE_DIRS}
  ${IGNITION-COMMON_INCLUDE_DIRS}
  ${IGNITION-MATHS_INCLUDE_DIRS}
  ${IGNITION-MSGS_INCLUDE_DIRS}
)

link_directories(
  ${gazebo_dev_LIBRARY_DIRS}
  ${GAZEBO_LIBRARY_DIRS}
  ${IGNITION-COMMON_LIBRARY_DIRS}
  ${IGNITION-MATHS_LIBRARY_DIRS}
  ${IGNITION-MSGS_LIBRARY_DIRS}
)

add_library(Hydrodynamics
  SHARED
    plugins/Gazebo.cc
    plugins/Geometry.cc
    plugins/PhysicalConstants.cc
    plugins/Physics.cc
    plugins/Utilities.cc
    plugins/Wavefield.cc
    plugins/WavefieldEntity.cc
)
target_link_libraries(Hydrodynamics
  ${Boost_LIBRARIES}
  ${GAZEBO_LIBRARIES}
)
target_compile_options(Hydrodynamics PRIVATE "-Wno-unknown-pragmas")
list(APPEND WAVE_GAZEBO_LIBRARIES_LIST Hydrodynamics)
ament_export_libraries(Hydrodynamics)

# WavefieldModelPlugin
add_library(WavefieldModelPlugin
  SHARED
  plugins/WavefieldModelPlugin.cc
)
target_link_libraries(WavefieldModelPlugin
  ${Boost_LIBRARIES}
  ${GAZEBO_LIBRARIES}
  ${WAVE_GAZEBO_LIBRARIES_LIST}
)
target_compile_options(WavefieldModelPlugin PRIVATE "-Wno-unknown-pragmas")
list(APPEND WAVE_GAZEBO_PLUGINS_LIST WavefieldModelPlugin)
ament_export_libraries(WavefieldModelPlugin)

# WavefieldVisualPlugin
add_library(WavefieldVisualPlugin
  SHARED
  plugins/WavefieldVisualPlugin.cc
)
target_link_libraries(WavefieldVisualPlugin
  ${Boost_LIBRARIES}
  ${catkin_LIBRARIES}
  ${GAZEBO_LIBRARIES}
  ${WAVE_GAZEBO_LIBRARIES_LIST}
)
target_compile_options(WavefieldVisualPlugin PRIVATE "-Wno-unknown-pragmas")
list(APPEND WAVE_GAZEBO_PLUGINS_LIST WavefieldVisualPlugin)
ament_export_libraries(WavefieldVisualPlugin)

# WavegaugePlugin
add_library(wavegauge_plugin
  SHARED
  plugins/wavegauge_plugin.cc
)
target_link_libraries(wavegauge_plugin
  WavefieldModelPlugin
  ${Boost_LIBRARIES}
  ${GAZEBO_LIBRARIES}
  ${WAVE_GAZEBO_LIBRARIES_LIST}
)
add_dependencies(wavegauge_plugin WavefieldModelPlugin)
target_compile_options(wavegauge_plugin PRIVATE "-Wno-unknown-pragmas")
list(APPEND WAVE_GAZEBO_PLUGINS_LIST wavegauge_plugin)
ament_export_libraries(wavegauge_plugin)

# add_library(WavefieldModelPlugin SHARED plugins/WavefieldModelPlugin.cc)
# ament_target_dependencies(WavefieldModelPlugin
# 	"gazebo_dev"
# 	"gazebo_ros"
# )
# ament_export_libraries(WavefieldModelPlugin)

# add_library(WavefieldVisualPlugin SHARED plugins/WavefieldVisualPlugin.cc)
# ament_target_dependencies(WavefieldVisualPlugin
# 	"gazebo_dev"
# 	"gazebo_ros"
# )
# ament_export_libraries(WavefieldVisualPlugin)

install(TARGETS 
  ${WAVE_GAZEBO_LIBRARIES_LIST}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(TARGETS
  ${WAVE_GAZEBO_PLUGINS_LIST}  
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin)

install(TARGETS
	DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY models worlds launch
  DESTINATION share/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
