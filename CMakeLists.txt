cmake_minimum_required(VERSION 3.5)
project(sick_safetyscanners2)

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
find_package(Boost REQUIRED COMPONENTS chrono date_time system)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
find_package(lifecycle_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(sick_safetyscanners_base REQUIRED)
find_package(sick_safetyscanners2_interfaces REQUIRED)

set(dependencies
  rclcpp
  rclcpp_lifecycle
  lifecycle_msgs
  sensor_msgs
  rclcpp_components
  sick_safetyscanners_base
  sick_safetyscanners2_interfaces
)

# Component

add_library(sick_safetyscanners2_component SHARED
  src/SickSafetyscannersRos2.cpp
  src/utils/MessageCreator.cpp)

target_include_directories(sick_safetyscanners2_component PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

ament_target_dependencies(sick_safetyscanners2_component ${dependencies})

target_link_libraries(sick_safetyscanners2_component
  sick_safetyscanners_base::sick_safetyscanners_base
  ${Boost_LIBRARIES})

rclcpp_components_register_nodes(sick_safetyscanners2_component "sick::SickSafetyscannersRos2")

install(TARGETS sick_safetyscanners2_component EXPORT sick_safetyscanners2_component)

# Executable

add_executable(sick_safetyscanners2_node 
  src/sick_safetyscanners2_node.cpp)

target_link_libraries(sick_safetyscanners2_node sick_safetyscanners2_component)

install(TARGETS sick_safetyscanners2_node
  EXPORT export_${PROJECT_NAME}
  DESTINATION lib/${PROJECT_NAME})

# Lifecycle component

add_library(sick_safetyscanners2_lifecycle_component SHARED
  src/SickSafetyscannersLifeCycle.cpp
  src/utils/MessageCreator.cpp)

target_include_directories(sick_safetyscanners2_lifecycle_component PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

ament_target_dependencies(sick_safetyscanners2_lifecycle_component ${dependencies})

target_link_libraries(sick_safetyscanners2_lifecycle_component
  sick_safetyscanners_base::sick_safetyscanners_base
  ${Boost_LIBRARIES})

rclcpp_components_register_nodes(sick_safetyscanners2_lifecycle_component "sick::SickSafetyscannersLifeCycle")

install(TARGETS sick_safetyscanners2_lifecycle_component EXPORT sick_safetyscanners2_lifecycle_component)

# Lifecycle Executable

add_executable(sick_safetyscanners2_lifecycle_node
  src/sick_safetyscanners2_lifecycle_node.cpp)

target_link_libraries(sick_safetyscanners2_lifecycle_node
  sick_safetyscanners2_lifecycle_component)

install(TARGETS sick_safetyscanners2_lifecycle_node
  EXPORT export_${PROJECT_NAME}
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
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

#TODO lookup what this does
# For message creation,if used in this message
ament_export_dependencies(${dependencies})

ament_package()
