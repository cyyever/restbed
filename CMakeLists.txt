cmake_minimum_required(VERSION 3.19)

project(
  restbed
  VERSION 4.7.0
  LANGUAGES CXX)

include(cmake/cyy_cmake/all.cmake)

message("                                                              ")
message("##############################################################")
message("#                                                            #")
message("#  Copyright 2013-2020, Corvusoft Ltd, All Rights Reserved.  #")
message("#                                                            #")
message("#  Restbed is dual-licensed requiring commerical entities    #")
message("#  to acquire a Corvusoft Permissive License, Contact        #")
message("#  sales@corvusoft.co.uk for details.                        #")
message("#                                                            #")
message("##############################################################")
message("                                                              ")

#
# Build Options
#
option(BUILD_TESTS "Build unit tests." ON)
option(BUILD_SSL "Build secure socket layer support." ON)

#
# Configuration
#
set(INCLUDE_DIR "${PROJECT_SOURCE_DIR}/source")
include_directories( ${INCLUDE_DIR} SYSTEM)
set(SOURCE_DIR "${PROJECT_SOURCE_DIR}/source/corvusoft/${PROJECT_NAME}")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")

# find_package(Boost REQUIRED)

if(BUILD_SSL)
  find_package(OpenSSL REQUIRED)
endif()

#
# Build
#
file(GLOB_RECURSE MANIFEST "${SOURCE_DIR}/*.cpp")

set(LIBRARY_NAME ${PROJECT_NAME})
add_library(${LIBRARY_NAME} ${MANIFEST})
set_target_properties(
  ${SHARED_LIBRARY_NAME}
  PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR}
             VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR})
# target_link_libraries(${LIBRARY_NAME} Boost::asio)
if(BUILD_SSL)
  target_link_libraries(${LIBRARY_NAME} OpenSSL::Crypto OpenSSL::SSL)
endif()

if(BUILD_TESTS)
  find_package(catch REQUIRED)

  enable_testing()
  add_subdirectory("${PROJECT_SOURCE_DIR}/test/unit")
  add_subdirectory("${PROJECT_SOURCE_DIR}/test/feature")
  add_subdirectory("${PROJECT_SOURCE_DIR}/test/regression")
  add_subdirectory("${PROJECT_SOURCE_DIR}/test/integration")
endif()

#
# Install
#
file(GLOB ARTIFACTS "${SOURCE_DIR}/*.hpp")

install(FILES "${INCLUDE_DIR}/${PROJECT_NAME}"
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include")
install(FILES ${ARTIFACTS}
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include/corvusoft/${PROJECT_NAME}")

install(
  TARGETS ${LIBRARY_NAME}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib COMPONENT library)
