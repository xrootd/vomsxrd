# CMake project file for VOMS-XRootD

cmake_minimum_required (VERSION 2.6)
project (vomsxrd)

if(NOT (CMAKE_VERSION VERSION_LESS "2.8.12"))
   cmake_policy(SET CMP0021 OLD)
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
 ${PROJECT_SOURCE_DIR}/src
 ${PROJECT_SOURCE_DIR}/cmake )

#-------------------------------------------------------------------------------
# Set HAVE_XRDCRYPTO if requested to do so
#-------------------------------------------------------------------------------
OPTION(HAVE_XRDCRYPTO "Do we have XrdCrypto headers?" TRUE)
IF(HAVE_XRDCRYPTO)
    ADD_DEFINITIONS(-DHAVE_XRDCRYPTO)
ENDIF(HAVE_XRDCRYPTO)

include_directories(${PROJECT_SOURCE_DIR}/src)

include( vomsxrdOSDefs )

if( NOT VOMSXRD_SUBMODULE )
  #-------------------------------------------------------------------------------
  # Generate the version header
  #-------------------------------------------------------------------------------
  add_custom_target(
    XrdSecgsiVOMSVers.hh
    ${CMAKE_SOURCE_DIR}/genversion.sh ${CMAKE_SOURCE_DIR} )
  # sigh, yet another ugly hack :(
  macro( add_library _target )
    _add_library( ${_target} ${ARGN} )
    add_dependencies( ${_target} XrdSecgsiVOMSVers.hh )
  endmacro()

  macro( add_executable _target )
    _add_executable( ${_target} ${ARGN} )
    add_dependencies( ${_target} XrdSecgsiVOMSVers.hh )
  endmacro()
endif()

#-------------------------------------------------------------------------------
# VOMS libs required
#-------------------------------------------------------------------------------
find_package( VOMS REQUIRED)
if( NOT VOMSXRD_SUBMODULE )
  find_package( XRootD CONFIG REQUIRED)
endif()

#-------------------------------------------------------------------------------
# Build in subdirectories
#-------------------------------------------------------------------------------
add_subdirectory( src )
 
#-------------------------------------------------------------------------------
# Summary
#-------------------------------------------------------------------------------
message( STATUS "----------------------------------------" )
message( STATUS "Installation path: " ${CMAKE_INSTALL_PREFIX} )
message( STATUS "Build type:        " ${CMAKE_BUILD_TYPE} )
message( STATUS "----------------------------------------" )

