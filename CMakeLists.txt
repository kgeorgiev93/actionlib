cmake_minimum_required(VERSION 2.8.3)

include(cmake/HunterGate.cmake)
HunterGate(
    URL "https://github.com/lsolanka/hunter/archive/v0.19.210-10.tar.gz"
    SHA1 "0a67d853099e5c33d0e37bc8e19b41a2b05a2c50"
)

project(actionlib)

hunter_add_package(ros-common_msgs)
hunter_add_package(ros-message_generation)
hunter_add_package(ros-ros)
hunter_add_package(ros-ros_comm)
hunter_add_package(ros-std_msgs)

find_package(catkin REQUIRED COMPONENTS actionlib_msgs message_generation roscpp rosunit std_msgs)
find_package(Boost CONFIG REQUIRED COMPONENTS thread)

include_directories(include ${catkin_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
link_directories(${catkin_LIBRARY_DIRS})

catkin_python_setup()

add_action_files(DIRECTORY action FILES Test.action TestRequest.action TwoInts.action)
generate_messages(DEPENDENCIES actionlib_msgs std_msgs)

set(PROJECT_INSTALLSPACE_LIBRARIES ros::actionlib)
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES actionlib
  CATKIN_DEPENDS actionlib_msgs message_runtime roscpp std_msgs
  DEPENDS "Boost COMPONENTS thread"
)

add_library(actionlib src/connection_monitor.cpp src/goal_id_generator.cpp)
target_link_libraries(actionlib ${catkin_LIBRARIES} Boost::thread)
add_dependencies(actionlib actionlib_gencpp)
target_include_directories(actionlib PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

catkin_install_python(PROGRAMS tools/axclient.py tools/axserver.py tools/dynamic_action.py tools/library.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}Targets
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
install(EXPORT ${PROJECT_NAME}Targets
    DESTINATION share/${PROJECT_NAME}/cmake
    NAMESPACE ros::
    FILE ${PROJECT_NAME}Targets.cmake)
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")

if(CATKIN_ENABLE_TESTING)
  find_package(rostest CONFIG REQUIRED)
  add_subdirectory(test)
endif()
