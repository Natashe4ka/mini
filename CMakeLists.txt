cmake_minimum_required(VERSION 3.22)
project(mini)

find_package(ament_cmake REQUIRED)


if(CMAKE_CXX_COMPILER_ID MATCHES "(GNU|Clang)")
  add_compile_options(-Wall -Wextra)
endif()

set(THIS_PACKAGE_INCLUDE_DEPENDS
  hardware_interface
  controller_interface
  realtime_tools
 # semantic_components
  sensor_msgs
  pluginlib
  rclcpp
  rclcpp_lifecycle
  
)

find_package(ament_cmake REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()

add_library(
  mini
  SHARED
  hardware/mini_hardware.cpp
  )

target_compile_features(mini PUBLIC cxx_std_17)
target_include_directories(mini PUBLIC
$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/hardware/include>
#$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/controller/include>
$<INSTALL_INTERFACE:include/mini>
)
ament_target_dependencies(
  mini PUBLIC
  ${THIS_PACKAGE_INCLUDE_DEPENDS}
 
)

target_compile_definitions(${PROJECT_NAME} PRIVATE "MINI_BUILDING_DLL")

pluginlib_export_plugin_description_file(hardware_interface mini.xml)

install(DIRECTORY launch DESTINATION share/${PROJECT_NAME}
  PATTERN "setup_assistant.launch" EXCLUDE)
install(DIRECTORY config DESTINATION share/${PROJECT_NAME})
install(FILES .setup_assistant DESTINATION share/${PROJECT_NAME})


install(
  DIRECTORY hardware/include/
  DESTINATION include/mini
)



install(TARGETS mini
  EXPORT export_mini
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

if(BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
endif()

## EXPORTS
ament_export_targets(export_mini HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})




ament_package()
