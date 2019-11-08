# set type specific output defaults
include(GNUInstallDirs)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
SET(CMAKE_Fortran_MODULE_DIRECTORY "${PROJECT_BINARY_DIR}/modules")

#################################################################
# Library target
#################################################################
set(LIB FLAP)
add_library(${LIB} 
        flap_command_line_arguments_group_t.f90
        flap_command_line_argument_t.F90
        flap_command_line_interface_t.F90
        flap.f90
        flap_object_t.f90
        flap_utils_m.f90
)
add_library(${NAMESPACE}${LIB} ALIAS ${LIB})

target_include_directories(${LIB}
    INTERFACE
        $<BUILD_INTERFACE:${CMAKE_Fortran_MODULE_DIRECTORY}>
)

set_target_properties(${LIB} PROPERTIES
    VERSION
        ${PROJECT_VERSION}
    SOVERSION
        ${PROJECT_VERSION_MAJOR}
)

TARGET_LINK_LIBRARIES(${LIB} 
        PUBLIC
                $<BUILD_INTERFACE:${FACE_LIBRARY}>
                $<BUILD_INTERFACE:${PENF_LIBRARY}>
                $<INSTALL_INTERFACE:FACE::FACE>
                $<INSTALL_INTERFACE:PENF::PENF>
)
IF(CMAKE_BUILD_TYPE MATCHES "DEBUG")
   TARGET_LINK_LIBRARIES(${LIB} INTERFACE -lgcov)
ENDIF()
TARGET_INCLUDE_DIRECTORIES(${LIB} 
        PUBLIC
                $<BUILD_INTERFACE:${FACE_INCLUDE_DIR}>
                $<BUILD_INTERFACE:${PENF_INCLUDE_DIR}>
        INTERFACE
        $<BUILD_INTERFACE:${CMAKE_INSTALL_FULL_INCLUDEDIR}/${LIB}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${LIB}>
)
ADD_DEPENDENCIES(${LIB} PENF FACE)

# installation and export of targets
install(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY}/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    COMPONENT Development
)

install(TARGETS ${LIB} EXPORT ${TARGETS_EXPORT_NAME}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT RuntimeLibraries
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT RuntimeLibraries
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    )

