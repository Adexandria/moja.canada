### Unit test ###
set(TESTUNIT "${LIBNAME}.test")

find_package(Boost COMPONENTS unit_test_framework REQUIRED)
if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
endif()

find_package(Turtle)
if(TURTLE_FOUND)
	include_directories(${Turtle_INCLUDE_PATH})
endif()

configure_file(../../templates/unittestdefinition.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/_unittestdefinition.cpp)

set(TEST_SRCS
    src/_unittestdefinition.cpp   
    src/treeyieldtabletests.cpp
    src/standgrowthcurvetests.cpp
    src/perdfactortests.cpp	
    src/volumetobiomassconvertertests.cpp
    src/smoothertests.cpp
    src/volumetobiomasscarbongrowthtests.cpp
    src/recordaccumulatortests.cpp
    src/recordaccumulatorintegrationtests.cpp
)

add_definitions(-DBOOST_LOG_DYN_LINK)
add_definitions(-DBOOST_ALL_DYN_LINK)

add_executable(${TESTUNIT} ${TEST_SRCS})
add_test(NAME ${LIBNAME} 
         WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
         COMMAND ${TESTUNIT} --result_code=yes --report_level=no)

target_link_libraries(
    ${TESTUNIT}
    ${LIBNAME}
    ${Boost_LIBRARIES}
    ${SYSLIBS}
    ${Moja_CORE}
	${Moja_FLINT}
    ${Moja_DATAREPOSITORY}
)

if(RUN_UNIT_TESTS_ON_BUILD)
    add_custom_command(TARGET ${TESTUNIT}
                       POST_BUILD COMMAND ${CMAKE_CTEST_COMMAND}
                       ARGS -C $<CONFIGURATION>)
endif()
### End unit test ###

### Begin PATH boilerplate for dependent libraries -- adapted from ###
### http://www.cmake.org/pipermail/cmake/2009-May/029464.html      ###
### This allows unit tests to run without having to manually add   ###
### dependent libraries to the system path.                        ###

# Include an entry for each library that needs to be in the system path.
find_path(POCO_BIN NAMES PocoFoundation.dll PocoFoundation64.dll
          PATHS ${POCO_PATH}/bin ${POCO_PATH}/bin64
          PATH_SUFFIXES $<CONFIGURATION>)

file(TO_NATIVE_PATH "${Boost_LIBRARY_DIR}" boost_lib)
file(TO_NATIVE_PATH "${POCO_BIN}" poco_bin)

# Determine which environment variable controls the locating of
# DLL's and set that variable.
if(WIN32)
    set(LD_VARNAME "PATH")
    set(LD_PATH "${boost_lib};${poco_bin};$ENV{PATH}")

    # IMPORTANT NOTE: The set_tests_properties(), below, internally
    # stores its name/value pairs with a semicolon delimiter.
    # because of this we must protect the semicolons in the path.
    string(REPLACE ";" "\\;" LD_PATH "${LD_PATH}")
else()
    set(LD_VARNAME "LD_LIBRARY_PATH")
    set(LD_PATH "${boost_lib}:$ENV{LD_LIBRARY_PATH}")
endif()

set_tests_properties(${LIBNAME} PROPERTIES ENVIRONMENT "${LD_VARNAME}=${LD_PATH}")
### End PATH boilerplate ###
