cmake_minimum_required(VERSION 3.12)
project(sparse_bundle_adjustment)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# Find ROS 2 and system dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED) # Replaces roscpp
find_package(Eigen3 REQUIRED)

# SuiteSparse fallback logic retained
find_package(suitesparse QUIET)
if(NOT SuiteSparse_FOUND)
  set(SuiteSparse_LIBRARIES blas lapack cholmod cxsparse)
endif()

# Define the shared library
add_library(sba 
  src/sba.cpp 
  src/spa.cpp 
  src/spa2d.cpp 
  src/csparse.cpp 
  src/proj.cpp 
  src/node.cpp 
  src/sba_file_io.cpp
)

# Apply definitions directly to the target
target_compile_definitions(sba PUBLIC SBA_CHOLMOD)

# Interface includes
target_include_directories(sba 
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    ${SuiteSparse_INCLUDE_DIRS}
)

# Link ROS 2 dependencies
ament_target_dependencies(sba PUBLIC rclcpp)

# Link system dependencies (using modern imported targets where available)
target_link_libraries(sba 
  PUBLIC 
    Eigen3::Eigen 
    ${SuiteSparse_LIBRARIES}
)

# Install headers
install(
  DIRECTORY include/${PROJECT_NAME}/
  DESTINATION include/${PROJECT_NAME}
)

# Install target and export it
install(
  TARGETS sba
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Export macros for downstream packages
ament_export_targets(export_${PROJECT_NAME})
ament_export_dependencies(rclcpp Eigen3)

# Finalize the ament package
ament_package()