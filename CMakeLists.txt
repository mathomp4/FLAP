#################################################################
# HEADER
#################################################################
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.11)
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE)

CMAKE_POLICY(SET CMP0022 NEW)

PROJECT(FLAP Fortran)

SET(${PROJECT_NAME}_VERSION 0.0.1)
SET(${PROJECT_NAME}_SOVERSION 1)
SET(LIB ${PROJECT_NAME})

SET(CMAKE_POSITION_INDEPENDENT_CODE ON)

SET(FLAP_CONFIG_MODULE ${CMAKE_CURRENT_LIST_DIR}/CMake/FLAPConfig.cmake.in)

#################################################################
# CONFIGURATION TYPES & BUILD MODE
#################################################################

SET(CMAKE_CONFIGURATION_TYPES DEBUG RELEASE)
IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE DEBUG CACHE STRING
      "Choose the type of build, options are: NONE DEBUG RELEASE"
      FORCE)

  SET_PROPERTY(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS NONE DEBUG RELEASE)
ENDIF(NOT CMAKE_BUILD_TYPE)

IF(NOT ${PROJECT_NAME}_ENABLE_TESTS)
    OPTION(${PROJECT_NAME}_ENABLE_TESTS "Enable/disable tests compilation" OFF)
ENDIF(NOT ${PROJECT_NAME}_ENABLE_TESTS)

#################################################################
# STATIC LIBRARIES
#################################################################
# Try to search first static libraries
IF(NOT ${BUILD_SHARED_LIBS})
    SET(CMAKE_FIND_LIBRARY_SUFFIXES ".a ${CMAKE_FIND_LIBRARY_SUFFIXES}")
    SET( HDF5_USE_STATIC_LIBRARIES ON)
ENDIF()


#################################################################
# FFLAGS depend on the compiler and the build type
#################################################################

GET_FILENAME_COMPONENT(Fortran_COMPILER_NAME ${CMAKE_Fortran_COMPILER} NAME)

IF(CMAKE_BUILD_TYPE STREQUAL "DEBUG")
#   SET(MACROS "${MACROS} -DDEBUG -Dmemcheck")
    ADD_DEFINITIONS(-DDEBUG)
    ADD_DEFINITIONS(-Dmemcheck)
ENDIF()

ADD_DEFINITIONS(-D${CMAKE_Fortran_COMPILER_ID})

message(STATUS "COMPILER INFO: ${CMAKE_Fortran_COMPILER_ID} - ${Fortran_COMPILER_NAME}")

IF (${CMAKE_Fortran_COMPILER_ID} STREQUAL "GNU" OR Fortran_COMPILER_NAME MATCHES "gfortran*")
  # gfortran 
  set(FORTRAN_FLAGS "-ffree-line-length-0 -cpp -Wimplicit-interface ${EXTRA_FLAGS} ")
  set (CMAKE_Fortran_FLAGS "${FORTRAN_FLAGS} ${MACROS} ${INCLUDES} " CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_RELEASE "-O3 ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_DEBUG   "-g -fbacktrace -fbounds-check -fprofile-arcs -ftest-coverage -Wimplicit-interface ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
ELSEIF (${CMAKE_Fortran_COMPILER_ID} STREQUAL "Intel" OR Fortran_COMPILER_NAME MATCHES "ifort*")
  # ifort (untested)
  set(FORTRAN_FLAGS "-r8 -fpp -W1 ${EXTRA_FLAGS} ")
  set (CMAKE_Fortran_FLAGS "${FORTRAN_FLAGS} ${MACROS} ${INCLUDES}" CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_RELEASE "-O3 ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_DEBUG   "-O0 -traceback -g -debug all -check all -ftrapuv -warn nointerfaces ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
# A partir de CMake 3.1
# -prof-gen:srcpos -prof-dir${PROJECT_BINARY_DIR}

ELSEIF (${CMAKE_Fortran_COMPILER_ID} STREQUAL "XL" OR Fortran_COMPILER_NAME MATCHES "xlf*")
  # xlf (untested)
  set(FORTRAN_FLAGS "-q64 -qrealsize=8 -qsuffix=f=f90:cpp=f90 -qxlf2003=polymorphic -Wl,-dy ${EXTRA_FLAGS} ")
  set (CMAKE_Fortran_FLAGS "${FORTRAN_FLAGS} ${MACROS} ${INCLUDES}" CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_RELEASE "-O3 -qstrict ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
  set (CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g -qfullpath -qkeepparm ${EXTRA_FLAGS} " CACHE STRING "" FORCE)
ELSE ()
  message ("No optimized Fortran compiler flags are known, we just try -O2...")
  set (CMAKE_Fortran_FLAGS_RELEASE "-O2")
  set (CMAKE_Fortran_FLAGS_DEBUG   "-O0 -g")
ENDIF ()

SET(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -Wl,--allow-multiple-definition")
SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--allow-multiple-definition")

message (STATUS "CMAKE_Fortran_COMPILER full path: " ${CMAKE_Fortran_COMPILER})
message (STATUS "CMAKE_Fortran_FLAGS: " ${CMAKE_Fortran_FLAGS})
message (STATUS "CMAKE_Fortran_FLAGS_RELEASE: " ${CMAKE_Fortran_FLAGS_RELEASE})
message (STATUS "CMAKE_Fortran_FLAGS_DEBUG: " ${CMAKE_Fortran_FLAGS_DEBUG})

#################################################################
# ENABLE TESTING
#################################################################

SET(BUILDNAME ${CMAKE_Fortran_COMPILER_ID}_${CMAKE_BUILD_TYPE}_MKL=${${PROJECT_NAME}_ENABLE_MKL} CACHE STRING "" )
IF(${PROJECT_NAME}_ENABLE_TESTS)
    ENABLE_TESTING()
    INCLUDE(CTest)
ENDIF()

#################################################################
# Add external projects
#################################################################

include(ExternalProject)
include(GNUInstallDirs)

EXTERNALPROJECT_Add(PENF
  PREFIX      ${CMAKE_CURRENT_BINARY_DIR}/PENF
  SOURCE_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/src/third_party/PENF
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}

  CMAKE_ARGS
    -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} 
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_INSTALL_LIBDIR=${CMAKE_INSTALL_LIBDIR}
    -DCMAKE_POSITION_INDEPENDENT_CODE=${CMAKE_POSITION_INDEPENDENT_CODE}
)
SET(PENF_LIBRARY ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}/libPENF.a)
SET(PENF_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include/PENF)


EXTERNALPROJECT_Add(FACE
  PREFIX      ${CMAKE_CURRENT_BINARY_DIR}/FACE
  SOURCE_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/src/third_party/FACE
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}

  CMAKE_ARGS
    -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} 
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
    -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_INSTALL_LIBDIR=${CMAKE_INSTALL_LIBDIR}
    -DCMAKE_POSITION_INDEPENDENT_CODE=${CMAKE_POSITION_INDEPENDENT_CODE}
)
SET(FACE_LIBRARY ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}/libFACE.a)
SET(FACE_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include/FACE)

#################################################################
# ADD SOURCE SUBDIRS
#################################################################

ADD_SUBDIRECTORY(src/lib)
IF(${PROJECT_NAME}_ENABLE_TESTS)
    EXTERNALPROJECT_Add(fortran_tester
      PREFIX      ${CMAKE_CURRENT_BINARY_DIR}/fortran_tester
      SOURCE_DIR  ${CMAKE_CURRENT_SOURCE_DIR}/src/third_party/fortran_tester
      INSTALL_DIR ${CMAKE_INSTALL_PREFIX}

      CMAKE_ARGS
        -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} 
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
        -DCMAKE_INSTALL_LIBDIR=${CMAKE_INSTALL_LIBDIR}
        -DCMAKE_POSITION_INDEPENDENT_CODE=${CMAKE_POSITION_INDEPENDENT_CODE}
    )
    SET(fortran_tester_LIBRARY ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}/libfortran_tester.a)
    SET(fortran_tester_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include/fortran_tester)

    ADD_SUBDIRECTORY(src/tests)
ENDIF()
