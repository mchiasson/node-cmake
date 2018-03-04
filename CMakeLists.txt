cmake_minimum_required(VERSION 3.5)

project(node)
include(ExternalProject)
include(ProcessorCount)

ProcessorCount(N)

set(NODE_VERSION 8.9.4)
set(NODE_LIB "${CMAKE_SHARED_LIBRARY_PREFIX}node${CMAKE_SHARED_LIBRARY_SUFFIX}.57")

ExternalProject_add(_${PROJECT_NAME}
    PREFIX /tmp/_${PROJECT_NAME}
    URL https://github.com/nodejs/node/archive/v${NODE_VERSION}.tar.gz
    URL_HASH SHA1=9593b1bce76ac9bc708e8f0c79fed0873f78f6d0

    CONFIGURE_COMMAND ./configure --prefix=<INSTALL_DIR> --shared
    BUILD_COMMAND make -j${N}
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND make install

    BUILD_BYPRODUCTS
        <INSTALL_DIR>/lib/${NODE_LIB}
)

ExternalProject_Get_Property(_${PROJECT_NAME} INSTALL_DIR)

add_library(${PROJECT_NAME} INTERFACE)
target_link_libraries(${PROJECT_NAME} INTERFACE
    $<BUILD_INTERFACE:${INSTALL_DIR}/lib/${NODE_LIB}>
    $<INSTALL_INTERFACE:lib/${NODE_LIB}>
)
target_include_directories(${PROJECT_NAME} INTERFACE
    $<BUILD_INTERFACE:${INSTALL_DIR}/include/node>
    $<INSTALL_INTERFACE:include/node>
)
add_dependencies(${PROJECT_NAME} _${PROJECT_NAME})

# Create the CMake version file.
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION ${NODE_VERSION}
  COMPATIBILITY AnyNewerVersion
)

set(targets_export_name "${PROJECT_NAME}Targets")

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
set(ConfigPackageLocation lib/cmake/${PROJECT_NAME})
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION ${ConfigPackageLocation}
)

# Install the generated CMake files.
install(EXPORT ${targets_export_name} NAMESPACE "${PROJECT_NAME}::" DESTINATION lib/cmake/${PROJECT_NAME})
install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  DESTINATION
    ${ConfigPackageLocation}
)
install(
    FILES
        ${INSTALL_DIR}/src/_node/src/aliased_buffer.h
        ${INSTALL_DIR}/src/_node/src/node_javascript.h
        ${INSTALL_DIR}/src/_node/src/node_internals.h
        ${INSTALL_DIR}/src/_node/src/util.h
        ${INSTALL_DIR}/src/_node/src/util-inl.h
        ${INSTALL_DIR}/src/_node/src/env.h
        ${INSTALL_DIR}/src/_node/src/env-inl.h
        ${INSTALL_DIR}/src/_node/src/node_perf_common.h
        ${INSTALL_DIR}/src/_node/src/node_debug_options.h
    DESTINATION 
        include/node
)
install(
    FILES
        ${INSTALL_DIR}/src/_node/src/tracing/trace_event.h
        ${INSTALL_DIR}/src/_node/src/tracing/trace_event_common.h
    DESTINATION 
        include/node/tracing
)

install(DIRECTORY ${INSTALL_DIR}/bin/     DESTINATION bin)
install(DIRECTORY ${INSTALL_DIR}/include/ DESTINATION include)
install(DIRECTORY ${INSTALL_DIR}/lib/     DESTINATION lib)
install(DIRECTORY ${INSTALL_DIR}/share/   DESTINATION share)
install(
    FILES
        ${INSTALL_DIR}/src/_${PROJECT_NAME}/LICENSE
    DESTINATION
        licenses/${PROJECT_NAME}/
)
