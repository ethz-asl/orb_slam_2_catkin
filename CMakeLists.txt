cmake_minimum_required(VERSION 2.8.3)
project(orb_slam_2_catkin)

add_definitions(-std=c++11)

find_package(catkin_simple REQUIRED)
catkin_simple()

include_directories(${Eigen_INCLUDE_DIRS})

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

ExternalProject_Add(orb_slam_2_src
  GIT_REPOSITORY https://github.com/raulmur/ORB_SLAM2.git
  UPDATE_COMMAND ""
  PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/fix_usleep.patch
  CONFIGURE_COMMAND
      # Configuring DBoW2
      mkdir -p ../orb_slam_2_src/Thirdparty/DBoW2/build && 
      cd ../orb_slam_2_src/Thirdparty/DBoW2/build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release &&
      # Configuring g2o
      mkdir -p ../../g2o/build &&
      cd ../../g2o/build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release &&
      # Uncompressing the vocabulary
      cd ../../../Vocabulary &&
      tar -xf ../../orb_slam_2_src/Vocabulary/ORBvoc.txt.tar.gz &&
      # Configuring the orb slam 2 build
      mkdir -p ../build &&
      cd ../build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release
  BUILD_COMMAND 
      # Building DBoW2
      cd ../orb_slam_2_src/Thirdparty/DBoW2/build/ &&
      make -j &&
      # Building g2o
      cd ../../g2o/build/ &&
      make -j &&
      # Building orb slam 2
      cd ../../../build/ &&
      make -j
  INSTALL_COMMAND
      cp ../orb_slam_2_src/lib/libORB_SLAM2${CMAKE_SHARED_LIBRARY_SUFFIX} ${CATKIN_DEVEL_PREFIX}/lib/ &&
      cp -r ../orb_slam_2_src/include ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2 &&
      mkdir -p ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/DBoW2/ &&
      cp -r ../orb_slam_2_src/Thirdparty/DBoW2/DBoW2 ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/DBoW2/ &&
      cp -r ../orb_slam_2_src/Thirdparty/DBoW2/DUtils ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/DBoW2/ &&
      mkdir -p ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/g2o/ &&
      cp -r ../orb_slam_2_src/Thirdparty/g2o/g2o/ ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/g2o/ &&
      cp -r ../orb_slam_2_src/Thirdparty/g2o/config.h ${CATKIN_DEVEL_PREFIX}/include/orb_slam_2/Thirdparty/g2o/
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cpp)
add_dependencies(${PROJECT_NAME} orb_slam_2_src)
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libORB_SLAM2${CMAKE_SHARED_LIBRARY_SUFFIX} ${catkin_LIBRARIES})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          CFG_EXTRAS orb_slam_2-extras.cmake)
