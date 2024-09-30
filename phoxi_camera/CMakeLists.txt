cmake_minimum_required(VERSION 3.8)
project(phoxi_camera)

set(CMAKE_BUILD_TYPE Debug)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies

set(DEPENDENCIES
    rclcpp
    rclpy
    pcl_msgs
    pcl_conversions
    sensor_msgs
    rclcpp_components
    cv_bridge
    std_msgs
    nav_msgs
    tf2_ros
    tf2
    tf2_geometry_msgs
    OpenCV
    std_srvs
    PhoXi
    rosidl_default_generators
    geometry_msgs
    phoxi_msgs
    tf2_eigen
    Eigen3
    PCL
    diagnostic_updater
    )



find_package(PhoXi REQUIRED CONFIG PATHS "$ENV{PHOXI_CONTROL_PATH}")
find_package(ament_cmake REQUIRED)
#find_package(Boost REQUIRED COMPONENTS system thread program_options)

foreach(Dependency IN ITEMS ${DEPENDENCIES})
  find_package(${Dependency} REQUIRED)
endforeach()

#set(INCLUDES
#    include/phoxi_camera/ros_interface.hpp
#    include/phoxi_camera/PhoXiInterface.h
#    include/phoxi_camera/PhoXiDeviceInformation.h
#    include/phoxi_camera/PhoXiException.h
#    include/phoxi_camera/RosConversions.h
#    include/phoxi_camera/PhoXiConversions.h
#)

set(INCLUDE_DIRECTORIES
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  #${Boost_INCLUDE_DIR}
  #${Boost_LIBRARIES}
  ${PHOXI_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIRS}
)

#message(Boost_LIBRARIES="${Boost_LIBRARIES}")
#message(Boost_INCLUDE_DIR="${Boost_INCLUDE_DIR}")
#message(Boost_FOUND="${Boost_FOUND}")



add_library(${PROJECT_NAME} SHARED src/RosInterface.cpp src/PhoXiInterface.cpp)
ament_target_dependencies(${PROJECT_NAME} ${DEPENDENCIES})
target_compile_features(${PROJECT_NAME} PUBLIC c_std_99 cxx_std_17)
target_include_directories(${PROJECT_NAME} PUBLIC ${INCLUDE_DIRECTORIES})
#target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} ${PHOXI_LIBRARY} )
target_link_libraries(${PROJECT_NAME} ${PHOXI_LIBRARY} )

add_executable(phoxi_camera_node src/phoxi_camera_node.cpp)
ament_target_dependencies(phoxi_camera_node ${DEPENDENCIES})
target_compile_features(phoxi_camera_node PUBLIC c_std_99 cxx_std_17)
target_include_directories(phoxi_camera_node PUBLIC ${INCLUDE_DIRECTORIES})
target_link_libraries(phoxi_camera_node
  ${PHOXI_LIBRARY}
  ${PCL_LIBRARIES}
  #${Boost_LIBRARIES}
  ${PROJECT_NAME}
  )
target_compile_options(phoxi_camera_node PRIVATE -g)

 install(TARGETS phoxi_camera_node DESTINATION lib/${PROJECT_NAME})


install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

install(TARGETS
phoxi_camera_node
DESTINATION lib/${PROJECT_NAME})

ament_export_dependencies(${DEPENDENCIES})
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})


if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()



ament_package()
