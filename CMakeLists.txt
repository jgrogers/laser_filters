cmake_minimum_required(VERSION 2.8.3)
project(laser_filters)

set (PKG_DEPENDS
  sensor_msgs
  roscpp
  tf
  filters
  message_filters
  laser_geometry
  pluginlib
  pcl_ros
)
set (BUILD_DEPENDS
  ${PKG_DEPENDS})
set (RUN_DEPENDS
  ${PKG_DEPENDS})

find_package (catkin REQUIRED ${BUILD_DEPENDS})
find_package(Boost REQUIRED COMPONENTS system signals)
find_package(PCL REQUIRED)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  
)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS ${RUN_DEPENDS}
  LIBRARIES pointcloud_filters laser_scan_filters
)
add_library(pointcloud_filters src/pointcloud_filters.cpp)
target_link_libraries(pointcloud_filters 
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
  ${PCL_LIBRARIES})

add_library(laser_scan_filters src/laser_scan_filters.cpp src/median_filter.cpp src/array_filter.cpp)
target_link_libraries(laser_scan_filters 
  ${catkin_LIBRARIES}
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES})

add_executable(scan_to_cloud_filter_chain src/scan_to_cloud_filter_chain.cpp)
target_link_libraries(scan_to_cloud_filter_chain 
  laser_scan_filters
  pointcloud_filters)

add_executable(scan_to_scan_filter_chain src/scan_to_scan_filter_chain.cpp)
target_link_libraries(scan_to_scan_filter_chain 
  laser_scan_filters)

add_executable(generic_laser_filter_node src/generic_laser_filter_node.cpp)
target_link_libraries(generic_laser_filter_node 
  laser_scan_filters)

#test scan filter chain filter
#add_executable(test_scan_filter_chain EXCLUDE_FROM_ALL test/test_scan_filter_chain.cpp)
#target_link_libraries(test_scan_filter_chain laser_scan_filters)

#rosbuild_add_gtest_build_flags(test_scan_filter_chain)
#rosbuild_add_rostest(test/test_scan_filter_chain.launch)

set(EXECUTABLES
  scan_to_cloud_filter_chain
  scan_to_scan_filter_chain
  generic_laser_filter_node
#  test_scan_filter_chain
  )

install(TARGETS ${EXECUTABLES} laser_scan_filters pointcloud_filters
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)

