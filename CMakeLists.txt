cmake_minimum_required(VERSION 3.0)

option(CHERI_PURE "Build for CHERI Sandbox ABI (disables exceptions and RTTI)" OFF)
option(DISABLE_EXCEPTIONS_RTTI "Disable exceptions and RTTI" OFF)
option(NO_UNWIND_LIBRARY "Use libcxxrt without an unwind library" OFF)

if (CHERI_PURE)
  set(DISABLE_EXCEPTIONS_RTTI ON)
  set(NO_UNWIND_LIBRARY ON)
endif()

if (DISABLE_EXCEPTIONS_RTTI)
  add_definitions(-DLIBCXXRT_NO_EXCEPTIONS_NO_RTTI)
endif()

enable_testing()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")

add_subdirectory(src)
IF(BUILD_TESTS)
add_subdirectory(test)
ENDIF()

