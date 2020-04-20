# CMake project file for VOMS-XRootD

cmake_minimum_required (VERSION 2.6)
project (vomsxrd)

if(NOT (CMAKE_VERSION VERSION_LESS "2.8.12"))
   cmake_policy(SET CMP0021 OLD)
endif()

#-------------------------------------------------------------------------------
# Set helper dir variables
#-------------------------------------------------------------------------------
if( VOMSXRD_SUBMODULE )
  set( VOMSXRD_SOURCE_DIR ${CMAKE_SOURCE_DIR}/src/XrdXrootdVoms )
  set( VOMSXRD_BINARY_DIR ${CMAKE_BINARY_DIR}/src/XrdXrootdVoms )
else()
  set( VOMSXRD_SOURCE_DIR ${CMAKE_SOURCE_DIR} )
  set( VOMSXRD_BINARY_DIR ${CMAKE_BINARY_DIR} )
endif()

# make sure that the default is a RELEASE
if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE RELEASE CACHE STRING
      "Choose the type of build, options are: None Debug Release."
      FORCE)
endif (NOT CMAKE_BUILD_TYPE)

if( NOT VOMSXRD_SUBMODULE )
  # Add uninstall target
  CONFIGURE_FILE(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)
  ADD_CUSTOM_TARGET(uninstall
    "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake")
endif()

set( CMAKE_MODULE_PATH
 ${VOMSXRD_SOURCE_DIR}/src
 ${VOMSXRD_SOURCE_DIR}/cmake )

#-------------------------------------------------------------------------------
# Set HAVE_XRDCRYPTO if requested to do so
#-------------------------------------------------------------------------------
OPTION(HAVE_XRDCRYPTO "Do we have XrdCrypto headers?" TRUE)
IF(HAVE_XRDCRYPTO)
    ADD_DEFINITIONS(-DHAVE_XRDCRYPTO)
ENDIF(HAVE_XRDCRYPTO)

include_directories(${PROJECT_SOURCE_DIR}/src ${VOMSXRD_SOURCE_DIR}/src)

include( vomsxrdOSDefs )

#-------------------------------------------------------------------------------
# Generate the version header
#-------------------------------------------------------------------------------
add_custom_target(
  XrdSecgsiVOMSVers.hh
  ${VOMSXRD_SOURCE_DIR}/genversion.sh ${VOMSXRD_SOURCE_DIR} )

#-------------------------------------------------------------------------------
# VOMS libs required
#-------------------------------------------------------------------------------
find_package( VOMS )
if( NOT VOMSXRD_SUBMODULE )
  find_package( XRootD CONFIG REQUIRED)
endif()

if( VOMS_FOUND )
  #-------------------------------------------------------------------------------
  # Make VOMS_FOUND global
  #-------------------------------------------------------------------------------
  SET(VOMS_FOUND  "${VOMS_FOUND}" CACHE INTERNAL "VOMS_FOUND")

  #-------------------------------------------------------------------------------
  # Build in subdirectories
  #-------------------------------------------------------------------------------
  add_subdirectory( src )
   
  #-------------------------------------------------------------------------------
  # Summary
  #-------------------------------------------------------------------------------
  message( STATUS "----------------------------------------" )
  message( STATUS "Installation path: " ${CMAKE_INSTALL_PREFIX} )
  message(t config --global push.default simple STATUS "Build type:        " ${CMAKE_BUILD_TYPE} )
  message( STATUS "----------------------------------------" )
elseif( NOT VOMSXRD_SUBMODULE )
  message( ERROR "Failed to build vomsxrd, voms development package not found." )
endif()
