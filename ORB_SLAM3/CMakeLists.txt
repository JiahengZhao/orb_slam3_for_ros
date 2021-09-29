cmake_minimum_required(VERSION 3.2)
project(orb_slam3)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3")
# set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=native")
# set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")

# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-deprecated -O3 -march=native ")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-deprecated -O3 -march=native")

add_compile_options(-std=c++11 -w -O3 -march=native) 
add_definitions(-DCOMPILEDWITHC11)

# Check C++11 or C++0x support
# include(CheckCXXCompilerFlag)
# CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
# CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
# if(COMPILER_SUPPORTS_CXX11)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
#    add_definitions(-DCOMPILEDWITHC11)
#    message(STATUS "Using flag -std=c++11.")
# elseif(COMPILER_SUPPORTS_CXX0X)
#    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
#    add_definitions(-DCOMPILEDWITHC0X)
#    message(STATUS "Using flag -std=c++0x.")
# else()
#    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
# endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

set(PACKAGE_DEPENDENCIES
  message_runtime
  message_filters
  rosbag
  roscpp
  roslaunch
  roslib
  sensor_msgs
  std_msgs
  tf
  cv_bridge
  image_transport
)

find_package(catkin REQUIRED COMPONENTS ${PACKAGE_DEPENDENCIES})

find_package(OpenCV 4.0)
if(NOT OpenCV_FOUND)
  find_package(OpenCV 3.0)
  if(NOT OpenCV_FOUND)
    message(FATAL_ERROR "OpenCV > 3.0 not found.")
  endif()
endif()
MESSAGE("OPENCV VERSION: " ${OpenCV_VERSION})

find_package(Eigen3 3.1.0 REQUIRED)
MESSAGE("Eigen 3 DIR: " ${EIGEN3_INCLUDE_DIR})

find_package(Pangolin REQUIRED)
MESSAGE("Pangolin DIR: " ${Pangolin_INCLUDE_DIRS})

find_package(realsense2)

find_library(g2o_LIBRARY NAMES g2o 
  PATHS "${CMAKE_INSTALL_PREFIX}/lib" REQUIRED NO_DEFAULT_PATH)
if(g2o_LIBRARY)
  MESSAGE("g2o lib is found at " ${g2o_LIBRARY})
else()
  MESSAGE("g2o lib is not found.")
endif()

find_library(DBoW2_LIBRARY NAMES DBoW2 
  PATHS "${CMAKE_INSTALL_PREFIX}/lib" REQUIRED NO_DEFAULT_PATH)
if(DBoW2_LIBRARY)
  MESSAGE("DBoW2 lib is found at " ${DBoW2_LIBRARY})
else()
  MESSAGE("DBoW2 lib is not found.")
endif()


find_package(Boost REQUIRED COMPONENTS system iostreams)

catkin_package(
  CATKIN_DEPENDS ${PACKAGE_DEPENDENCIES}
 INCLUDE_DIRS "."
 LIBRARIES ${PROJECT_NAME}
 DEPENDS EIGEN3 Boost
)



include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${PROJECT_SOURCE_DIR}/include/CameraModels
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
${PROJECT_SOURCE_DIR}/../Thirdparty/
${catkin_INCLUDE_DIRS}
# ${PROJECT_SOURCE_DIR}/../Thirdparty/g2o
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

set(SRC_TO_REMOVE "${PROJECT_SOURCE_DIR}/src/PnPsolver.cc;
                ${PROJECT_SOURCE_DIR}/include/ORBVocabulary.h;
                ${PROJECT_SOURCE_DIR}/include/PnPsolver.h;
                ${PROJECT_SOURCE_DIR}/src/ros_*.cc;
                ${PROJECT_SOURCE_DIR}/src/AR/*.*")
# message("SRC_TO_REMOVE:" ${SRC_TO_REMOVE})
file(GLOB_RECURSE ALL_SRCS "${PROJECT_SOURCE_DIR}/src/*.cc"
                     "${PROJECT_SOURCE_DIR}/src/*.cpp")
list(REMOVE_ITEM ALL_SRCS ${SRC_TO_REMOVE})
# message("Remove ALL_SRCS:" ${ALL_SRCS})
add_library(${PROJECT_NAME} SHARED ${ALL_SRCS})

# list(APPEND ${PROJECT_NAME} ${g2o_LIBRARY} ${DBoW2_LIBRARY})
# add_library(${PROJECT_NAME} SHARED
# src/Atlas.cc
# src/Converter.cc
# src/Frame.cc
# src/FrameDrawer.cc
# src/G2oTypes.cc
# src/ImuTypes.cc
# src/Initializer.cc
# src/KeyFrame.cc
# src/KeyFrameDatabase.cc
# src/LocalMapping.cc
# src/LoopClosing.cc
# src/Map.cc
# src/MapDrawer.cc
# src/MapPoint.cc
# src/MLPnPsolver.cpp
# src/OptimizableTypes.cpp
# src/Optimizer.cc
# src/ORBextractor.cc
# src/ORBmatcher.cc
# # src/PnPsolver.cc
# src/Sim3Solver.cc
# src/System.cc
# src/Tracking.cc
# src/TwoViewReconstruction.cc
# src/Viewer.cc
# src/CameraModels/KannalaBrandt8.cpp
# src/CameraModels/Pinhole.cpp
# include/Atlas.h
# include/Config.h
# include/Converter.h
# include/Frame.h
# include/FrameDrawer.h
# include/G2oTypes.h
# include/ImuTypes.h
# include/Initializer.h
# include/KeyFrame.h
# include/KeyFrameDatabase.h
# include/LocalMapping.h
# include/LoopClosing.h
# include/Map.h
# include/MapDrawer.h
# include/MapPoint.h
# include/MLPnPsolver.h
# include/OptimizableTypes.h
# include/Optimizer.h
# include/ORBextractor.h
# include/ORBmatcher.h
# # include/ORBVocabulary.h
# # include/PnPsolver.h
# include/Sim3Solver.h
# include/System.h
# include/Tracking.h
# include/TwoViewReconstruction.h
# include/Viewer.h
# include/CameraModels/GeometricCamera.h
# include/CameraModels/KannalaBrandt8.h
# include/CameraModels/Pinhole.h
# )

# add_subdirectory(Thirdparty/g2o)
set(LIBS ${PROJECT_NAME} 
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${g2o_LIBRARY} 
${DBoW2_LIBRARY}
-lboost_serialization
-lcrypto
${catkin_LIBRARIES}
# -lboost_system
)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
# ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
# ${PROJECT_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
${g2o_LIBRARY} 
${DBoW2_LIBRARY}
-lboost_serialization
-lcrypto
${catkin_LIBRARIES}
)

### Build examples

# RGB-D examples
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)

add_executable(rgbd_tum
Examples/RGB-D/rgbd_tum.cc)
target_link_libraries(rgbd_tum ${PROJECT_NAME})


# Stereo examples
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)

# add_executable(stereo_kitti
# Examples/Stereo/stereo_kitti.cc)
# target_link_libraries(stereo_kitti ${PROJECT_NAME})

# add_executable(stereo_euroc
# Examples/Stereo/stereo_euroc.cc)
# target_link_libraries(stereo_euroc ${PROJECT_NAME})

# add_executable(stereo_tum_vi
# Examples/Stereo/stereo_tum_vi.cc)
# target_link_libraries(stereo_tum_vi ${PROJECT_NAME})

# Monocular examples
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)

# add_executable(mono_tum
# Examples/Monocular/mono_tum.cc)
# target_link_libraries(mono_tum ${PROJECT_NAME})

# add_executable(mono_kitti
# Examples/Monocular/mono_kitti.cc)
# target_link_libraries(mono_kitti ${PROJECT_NAME})

# add_executable(mono_euroc
# Examples/Monocular/mono_euroc.cc)
# target_link_libraries(mono_euroc ${PROJECT_NAME})

# add_executable(mono_tum_vi
# Examples/Monocular/mono_tum_vi.cc)
# target_link_libraries(mono_tum_vi ${PROJECT_NAME})

# Monocular-Inertial examples
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular-Inertial)

# add_executable(mono_inertial_euroc
# Examples/Monocular-Inertial/mono_inertial_euroc.cc)
# target_link_libraries(mono_inertial_euroc ${PROJECT_NAME})

# add_executable(mono_inertial_tum_vi
# Examples/Monocular-Inertial/mono_inertial_tum_vi.cc)
# target_link_libraries(mono_inertial_tum_vi ${PROJECT_NAME})

# Stereo-Inertial examples
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo-Inertial)

# add_executable(stereo_inertial_euroc
# Examples/Stereo-Inertial/stereo_inertial_euroc.cc)
# target_link_libraries(stereo_inertial_euroc ${PROJECT_NAME})

# add_executable(stereo_inertial_tum_vi
# Examples/Stereo-Inertial/stereo_inertial_tum_vi.cc)
# target_link_libraries(stereo_inertial_tum_vi ${PROJECT_NAME})


# add_executable(RGBD_ros src/ros_rgbd.cc)
add_executable(RGBD_ros src/ros_rgbd.cc)
target_link_libraries(RGBD_ros ${PROJECT_NAME})
# target_link_libraries(RGBD_ros ${LIBS})


install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(TARGETS RGBD_ros ${PROJECT_NAME}
RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
LIBRARY DESTINATION lib)
