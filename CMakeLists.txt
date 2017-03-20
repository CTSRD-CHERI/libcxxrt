cmake_minimum_required(VERSION 3.0)

option(CHERI_PURE "Build for CHERI Sandbox ABI (disables exceptions and RTTI)" OFF)

if (CHERI_PURE)
  set(ENABLE_EXCEPTIONS_RTTI OFF)
else()
  set(ENABLE_EXCEPTIONS_RTTI ON)
endif()

if (NOT ENABLE_EXCEPTIONS_RTTI)
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

