cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

project(path_planning_tests LANGUAGES CXX)

# ---- Options ----

option(ENABLE_TEST_COVERAGE "Enable test coverage" OFF)
option(TEST_INSTALLED_VERSION "Test the version found by find_package" OFF)

# --- Import tools ----

include(../cmake/tools.cmake)

# ---- Dependencies ----

include(../cmake/CPM.cmake)

CPMAddPackage("gh:onqtam/doctest#2.4.5")
CPMAddPackage("gh:TheLartians/Format.cmake@1.7.0")

if(TEST_INSTALLED_VERSION)
  find_package(path_planning REQUIRED)
else()
  CPMAddPackage(NAME path_planning SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)
endif()

# ---- Create binary ----

file(GLOB sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)
add_executable(path_planning_tests ${sources})
target_link_libraries(path_planning_tests doctest::doctest path_planning::path_planning)
set_target_properties(path_planning_tests PROPERTIES CXX_STANDARD 17)

# enable compiler warnings
if(NOT TEST_INSTALLED_VERSION)
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(path_planning PUBLIC -Wall -Wpedantic -Wextra -Werror)
  elseif(MSVC)
    target_compile_options(path_planning PUBLIC /W4 /WX)
    target_compile_definitions(path_planning_tests PUBLIC DOCTEST_CONFIG_USE_STD_HEADERS)
  endif()
endif()

# ---- Add path_planning_tests ----

enable_testing()

# Note: doctest and similar testing frameworks can automatically configure CMake tests. For other
# testing frameworks add the tests target instead: add_test(NAME path_planning_tests COMMAND path_planning_tests)

include(${doctest_SOURCE_DIR}/scripts/cmake/doctest.cmake)
doctest_discover_tests(path_planning_tests)

# ---- code coverage ----

if(ENABLE_TEST_COVERAGE)
  target_compile_options(path_planning PUBLIC -O0 -g -fprofile-arcs -ftest-coverage)
  target_link_options(path_planning PUBLIC -fprofile-arcs -ftest-coverage)
endif()
