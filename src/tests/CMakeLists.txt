#################################################################
# TESTS
#################################################################

FILE(GLOB_RECURSE TESTS_SRC *.f90 *.F90)
SET(TESTS_SRC ${TESTS_SRC} PARENT_SCOPE)

#################################################################
# EXTERNAL LIBRARIES
#################################################################

FOREACH(TEST_SRC ${TESTS_SRC})
    GET_FILENAME_COMPONENT(EXE_NAME ${TEST_SRC} NAME_WE)
    ADD_EXECUTABLE(${EXE_NAME} ${TEST_SRC})
    TARGET_LINK_LIBRARIES(${EXE_NAME} PRIVATE ${LIB} ${fortran_tester_LIBRARY} FLAP)
    TARGET_INCLUDE_DIRECTORIES(${EXE_NAME} PRIVATE ${fortran_tester_INCLUDE_DIR})
    ADD_DEPENDENCIES(${EXE_NAME} fortran_tester)
    ADD_TEST(${EXE_NAME}_TEST ${EXECUTABLE_OUTPUT_PATH}/${EXE_NAME})
ENDFOREACH()



