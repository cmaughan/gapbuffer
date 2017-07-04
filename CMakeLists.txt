cmake_minimum_required(VERSION 3.1)

SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)

set (CMAKE_CXX_STANDARD 11)
set (CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download" )
execute_process(COMMAND "${CMAKE_COMMAND}" --build .
  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/googletest-download" )
 
# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
 
# Add googletest directly to our build. This adds
# the following targets: gtest, gtest_main, gmock
# and gmock_main
add_subdirectory("${CMAKE_BINARY_DIR}/googletest-src"
                 "${CMAKE_BINARY_DIR}/googletest-build")

# Gap buffer and tests 
SET(SOURCES
    gap_buffer.test.cpp
    gap_buffer.h
)

enable_testing()

project(gapbuffer_tests)
# Now simply link against gtest or gtest_main as needed. Eg
add_executable(gapbuffer_tests ${SOURCES})
add_dependencies(gapbuffer_tests gtest)
target_link_libraries(gapbuffer_tests gtest gtest_main)
add_test(gapbuffer_tests gapbuffer_tests)


