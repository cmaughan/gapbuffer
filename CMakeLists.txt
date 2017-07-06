cmake_minimum_required(VERSION 3.1)

SET_PROPERTY(GLOBAL PROPERTY USE_FOLDERS ON)

set (CMAKE_CXX_STANDARD 11)
set (CMAKE_EXPORT_COMPILE_COMMANDS ON)

if (CMAKE_BUILD_TYPE MATCHES Coverage)
    set (CMAKE_BUILD_TYPE "Debug")
    set (PROJECT_COVERAGE ON)
endif()

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
    README.md
)

# Make a test project
enable_testing()

project(gapbuffer_tests)
add_definitions(-D_SCL_SECURE_NO_WARNINGS)
add_executable(gapbuffer_tests ${SOURCES})
add_dependencies(gapbuffer_tests gtest)
target_link_libraries(gapbuffer_tests gtest gtest_main)
add_test(gapbuffer_tests gapbuffer_tests)

# Setup coverage
if(PROJECT_COVERAGE)
  message(STATUS "gapbuffer_tests coverage build")

  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -coverage")

  find_program(LCOV lcov)
  if(NOT LCOV)
    message(FATAL_ERROR "Executable lcov not found")
  endif()
  set(LCOV_ARGS
    --quiet
    --directory ${PROJECT_SOURCE_DIR}
    --rc lcov_branch_coverage=1
  )

  if(DEFINED ENV{GCOV})
    set(GCOV_TOOL $ENV{GCOV})
  else()
    set(GCOV_TOOL gcov)
  endif()
  find_program(GCOV ${GCOV_TOOL})
  if (NOT GCOV)
    message(FATAL_ERROR "Executable $ENV{GCOV} not found")
  endif()
  set(LCOV_ARGS ${LCOV_ARGS} --gcov-tool ${GCOV})

  find_program(GENHTML genhtml)
  if(NOT GENHTML)
    message(FATAL_ERROR "Executable genhtml not found")
  endif()

  add_custom_target(gen-cov
    DEPENDS gapbuffer_tests
    COMMENT "Generate coverage information"

    # Initialize coverage generation
    COMMAND ${LCOV} ${LCOV_ARGS} --zerocounters
    COMMAND ${LCOV} ${LCOV_ARGS} --initial --capture --no-external --derive-func-data --output-file coverage-base.info

    # Run and extract
    COMMAND ${CMAKE_COMMAND} --build . --target test
    COMMAND ${LCOV} ${LCOV_ARGS} --capture --no-external --derive-func-data --output-file coverage-tests.info
    COMMAND ${LCOV} ${LCOV_ARGS} --add-tracefile coverage-base.info --add-tracefile coverage-tests.info --output-file coverage.info
    COMMAND ${LCOV} ${LCOV_ARGS} --remove coverage.info '*.test.?pp' '*/gtest/*' '/user/*' --output-file coverage.info
    COMMAND ${LCOV} ${LCOV_ARGS} --list coverage.info
    COMMAND ${LCOV} ${LCOV_ARGS} --summary coverage.info
  )

  add_custom_command(TARGET gen-cov POST_BUILD
    COMMENT "Open ${CMAKE_BINARY_DIR}/cov/index.html in your browser"
    COMMAND ${GENHTML} --rc lcov_branch_coverage=1 -q --demangle-cpp --legend --output-directory cov coverage.info
  )
endif()
