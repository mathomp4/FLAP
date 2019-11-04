#################################################################
# Search F90 files recursively in all subdirs
#################################################################

FILE(GLOB_RECURSE LIB_SRC *.f90 *.F90 *.c)
SET(LIB_SRC ${LIB_SRC} PARENT_SCOPE)

#################################################################
# Library target
#################################################################
ADD_LIBRARY(FLAP 
        flap_command_line_arguments_group_t.f90
        flap_command_line_argument_t.F90
        flap_command_line_interface_t.F90
        flap.f90
        flap_object_t.f90
        flap_utils_m.f90
)
SET_TARGET_PROPERTIES(FLAP PROPERTIES 
        Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/modules
)

INCLUDE(GNUInstallDirs)

TARGET_LINK_LIBRARIES(FLAP 
        PUBLIC
                $<BUILD_INTERFACE:${FACE_LIBRARY}>
                $<BUILD_INTERFACE:${PENF_LIBRARY}>
                $<INSTALL_INTERFACE:FACE::FACE>
                $<INSTALL_INTERFACE:PENF::PENF>
)
IF(CMAKE_BUILD_TYPE MATCHES "DEBUG")
        TARGET_LINK_LIBRARIES(FLAP INTERFACE -lgcov)
ENDIF()
TARGET_INCLUDE_DIRECTORIES(FLAP 
        PUBLIC
                $<BUILD_INTERFACE:${FACE_INCLUDE_DIR}>
                $<BUILD_INTERFACE:${PENF_INCLUDE_DIR}>
        INTERFACE
                $<BUILD_INTERFACE:${CMAKE_INSTALL_FULL_INCLUDEDIR}/FLAP>
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/FLAP>
)
ADD_DEPENDENCIES(FLAP PENF FACE)

INSTALL(TARGETS FLAP
        EXPORT FLAP-export
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
INSTALL(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/modules/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/FLAP)
INSTALL(EXPORT FLAP-export
        FILE
                FLAPTargets.cmake
        NAMESPACE
                FLAP::
        DESTINATION
                ${CMAKE_INSTALL_LIBDIR}/cmake/FLAP
)
INCLUDE(CMakePackageConfigHelpers)
configure_package_config_file(${PROJECT_SOURCE_DIR}/CMake/FLAPConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/FLAPConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/FLAP
)

INSTALL(FILES 
        ${CMAKE_CURRENT_BINARY_DIR}/FLAPConfig.cmake 
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/FLAP
)
EXPORT(EXPORT FLAP-export FILE ${CMAKE_CURRENT_BINARY_DIR}/FLAPConfig.cmake NAMESPACE FLAP::)
EXPORT(PACKAGE FLAP)
