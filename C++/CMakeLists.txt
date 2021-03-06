CMAKE_MINIMUM_REQUIRED(VERSION 1.6)
PROJECT(Diffusion-MRI)

INCLUDE_REGULAR_EXPRESSION(".*")

# Tell CMake's FIND_PACKAGE command where to look for our Find<pkg>
# modules.
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${DW-MRI_SOURCE_DIR})

IF(NOT WIN32)
  OPTION(BUILD_SHARED_LIBS "Build Application with shared libraries." OFF)
ENDIF(NOT WIN32)

# Don't build anything unless the version of CMake is high enough.
# The matching ELSE/ENDIF should be the last lines in the file.
IF("${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" GREATER 1.5)

FIND_PACKAGE(VXL)
IF(VXL_FOUND)
  INCLUDE(${VXL_CMAKE_DIR}/UseVXL.cmake)
ENDIF(VXL_FOUND)
INCLUDE_DIRECTORIES( ${VXL_VNL_INCLUDE_DIR} )

OPTION(BUILD_REC_SHARED "Build the Reconstructor libraries shared." OFF)
IF(BUILD_REC_SHARED)
  SET(REC_LIBRARY_TYPE SHARED)
ELSE(BUILD_REC_SHARED)
  SET(REC_LIBRARY_TYPE STATIC)
ENDIF(BUILD_REC_SHARED)

SET(RECONSTRUCTION_SRCS
    reconstructor.cc
    qbi_reconstructor.cc
    dot_reconstructor.cc
    mow_reconstructor.cc
    spherical_harmonics.cc
    special_functions.c
    nnls.cc
    bits_utils.c
    fltfile_utils.cc
    vnl_utils.cc
)

IF(WIN32)
  ADD_DEFINITIONS(-DWIN32 -D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE)
ELSE(WIN32)
  OPTION(BUILD_INI_SHARED "Build the port_ini libraries shared." OFF)
  IF(BUILD_INI_SHARED)
    SET(INI_LIBRARY_TYPE SHARED)
  ELSE(BUILD_INI_SHARED)
    SET(INI_LIBRARY_TYPE STATIC)
  ENDIF(BUILD_INI_SHARED)
  ADD_LIBRARY(port_ini ${INI_LIBRARY_TYPE} port_ini.c)
ENDIF(WIN32)

if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

# generate STATIC library
ADD_LIBRARY(reconstructor ${REC_LIBRARY_TYPE} ${RECONSTRUCTION_SRCS} ${RECONSTRUCTION_HDRS})
TARGET_LINK_LIBRARIES(reconstructor vnl_algo vnl vcl)
IF(UNIX)
    TARGET_LINK_LIBRARIES(reconstructor m port_ini)
ENDIF(UNIX)

ADD_EXECUTABLE(reconstruction reconstruction.cc)
TARGET_LINK_LIBRARIES(reconstruction reconstructor)
INSTALL_TARGETS(/bin reconstruction)

ADD_EXECUTABLE(nnls_test nnls_test.cc nnls.cc)
ADD_EXECUTABLE(vnl_nnls_test vnl_nnls_test.cc vnl_utils.cc nnls.cc)
TARGET_LINK_LIBRARIES(vnl_nnls_test vnl)

#make install
SET(CMAKE_INSTALL_PREFIX ${DW-MRI_SOURCE_DIR})

# If the version of CMake was too old, complain and build nothing.
# These should be the last lines in this file.
ELSE("${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" GREATER 1.5)
  MESSAGE(SEND_ERROR
          "This version of CMake is too old to build InsightApplications.  "
          "Please upgrade to at least CMake 1.6.")
ENDIF("${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" GREATER 1.5)
