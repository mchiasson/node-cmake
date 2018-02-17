cmake_minimum_required(VERSION 3.5)

project(node)
include(ExternalProject)
include(ProcessorCount)

ProcessorCount(N)

set(NODE_VERSION 9.5.0)
set(NODE_LIB "${CMAKE_INSTALL_PREFIX}/lib/${CMAKE_SHARED_LIBRARY_PREFIX}node${CMAKE_SHARED_LIBRARY_SUFFIX}.59")


ExternalProject_add(node_project
    URL https://github.com/nodejs/node/archive/v${NODE_VERSION}.tar.gz
    URL_MD5 2ad41c6743ebddc9628bb7f7e9e89106

    CONFIGURE_COMMAND  <SOURCE_DIR>/configure --prefix=${CMAKE_INSTALL_PREFIX} --shared
    BUILD_COMMAND make -j${N}
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND make install

    BUILD_BYPRODUCTS
        ${NODE_LIB}
)

add_library(node INTERFACE)
target_link_libraries(node INTERFACE "${NODE_LIB}")
target_include_directories(node INTERFACE "${CMAKE_INSTALL_PREFIX}/include")
add_dependencies(node node_project)


# Create the CMake version file.
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/nodeConfigVersion.cmake"
  VERSION ${NODE_VERSION}
  COMPATIBILITY AnyNewerVersion
)

set(targets_export_name "nodeTargets")

install(TARGETS ${PROJECT_NAME}
      EXPORT ${targets_export_name}
      LIBRARY DESTINATION lib COMPONENT Runtime
      ARCHIVE DESTINATION lib COMPONENT Development
      RUNTIME DESTINATION bin COMPONENT Runtime
      PUBLIC_HEADER DESTINATION include COMPONENT Development
      BUNDLE DESTINATION bin COMPONENT Runtime
)
 
# Create the Config file.
include(CMakePackageConfigHelpers)
set(ConfigPackageLocation lib/cmake/node)
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/nodeConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/nodeConfig.cmake"
  INSTALL_DESTINATION ${ConfigPackageLocation}
)

# Install the generated CMake files.
install(EXPORT ${targets_export_name} NAMESPACE "node::" DESTINATION lib/cmake/node)
install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/nodeConfigVersion.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/nodeConfig.cmake"
  DESTINATION ${ConfigPackageLocation}
)
