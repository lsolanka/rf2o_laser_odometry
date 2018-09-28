CMAKE_MINIMUM_REQUIRED(VERSION 3.3)

include(cmake/HunterGate.cmake)
HunterGate(
    URL "https://github.com/lsolanka/hunter/archive/v0.22.4-rosdevel17.tar.gz"
    SHA1 "7f07def9fe1f42e676b6e9eabfb808d91f18525e"
    LOCAL
)

# Require C++17

PROJECT(rf2o_laser_odometry VERSION 0.0.0)
set(CATKIN_BUILD_BINARY_PACKAGE ON)

if(${CMAKE_VERSION} VERSION_LESS "3.8.0")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
else()
	set(CMAKE_CXX_STANDARD 17)
endif()

hunter_add_package(Boost COMPONENTS system)
hunter_add_package(catkin)
hunter_add_package(Eigen)
hunter_add_package(MRPT)
hunter_add_package(ros_comm)
hunter_add_package(ros_common_msgs)
hunter_add_package(ros_tf)

if(HUNTER_ENABLED)
    set(Boost_LIBRARIES Boost::boost Boost:system)
    set(EIGEN_LIBRARIES Eigen3::Eigen)
endif()

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS  
  roscpp
  rospy
  nav_msgs
  sensor_msgs
  std_msgs
  tf  
)

## System dependencies are found with CMake's conventions
find_package(Boost CONFIG REQUIRED COMPONENTS system)
#find_package(cmake_modules REQUIRED)
find_package(Eigen3 CONFIG REQUIRED)

find_package(MRPT CONFIG REQUIRED)
MESSAGE(STATUS "Found MRPT: " ${MRPT_VERSION})
IF(MRPT_VERSION VERSION_LESS 1.9.9)
        # MRPT<2.0
        find_package(MRPT CONFIG REQUIRED base obs maps slam)
ELSE()
        # MRPT>=2.0
        find_package(MRPT CONFIG REQUIRED obs maps slam poses core)
ENDIF()


#include_directories(${MRPT_INCLUDE_DIRS})
#MESSAGE( STATUS "MRPT_INCLUDE_DIRS: " ${MRPT_INCLUDE_DIRS})
#link_directories(${MRPT_LIBRARY_DIRS})
#MESSAGE( STATUS "MRPT_LIBRARY_DIRS: " ${MRPT_LIBS})



###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
 INCLUDE_DIRS include
 LIBRARIES laser_odometry
 CATKIN_DEPENDS nav_msgs roscpp sensor_msgs std_msgs tf
 #DEPENDS system_lib
)

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(include)

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${EIGEN_INCLUDE_DIRS}
  ${MRPT_INCLUDE_DIRS}
)

## Declare a cpp executable
add_executable(rf2o_laser_odometry_node src/CLaserOdometry2D.cpp)

## Specify libraries to link a library or executable target against
target_link_libraries(rf2o_laser_odometry_node
   ${catkin_LIBRARIES}
   ${Boost_LIBRARIES}
   ${EIGEN_LIBRARIES}
   ${MRPT_LIBS}
)

install(TARGETS rf2o_laser_odometry_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# CPack
if(CMAKE_CROSSCOMPILING)
    if("${CMAKE_LIBRARY_ARCHITECTURE}" STREQUAL "arm-linux-gnueabihf")
        set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "armhf")
    else()
        message(FATAL_ERROR
            "CMAKE_LIBRARY_ARCHITECTURE is not set or not supported: "
            "'${CMAKE_LIBRARY_ARCHITECTURE}'")
    endif()
endif()
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_CONTACT "lsolanka AT gmail.com")
set(CPACK_PACKAGE_VENDOR "Lukas Solanka")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_FILE_NAME
    "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
set(CPACK_PACKAGING_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")
set(CPACK_GENERATOR "DEB")

include(CPack)
