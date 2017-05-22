
set(CXXTEST_SOURCES
    test.cc
    test_guard.cc
   )
if(NOT DISABLE_EXCEPTIONS_RTTI)
  list(APPEND CXXTEST_SOURCES test_exception.cc test_typeinfo.cc)
endif()


if(NOT NO_SHARED)
    add_executable(system-test ${CXXTEST_SOURCES})

    # Generating excpected output with system-test
    add_custom_target(test-expected-output ALL
                      COMMAND system-test > ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log 2>&1
                      DEPENDS system-test)
endif()

# It appears to be impossible to install libgcc_s.a on Ubuntu because they don't know how to package software
# Let's add a cmake flag to allow disabling this test
if(NOT NO_STATIC_TEST)
    add_executable(cxxrt-test-static ${CXXTEST_SOURCES})
    set_property(TARGET cxxrt-test-static PROPERTY LINK_FLAGS "-nodefaultlibs -static")
    target_link_libraries(cxxrt-test-static cxxrt-static gcc_s pthread ${CMAKE_DL_LIBS} c compiler_rt)
endif()

if(NOT NO_SHARED)
    add_executable(cxxrt-test-shared ${CXXTEST_SOURCES})
    set_property(TARGET cxxrt-test-shared PROPERTY LINK_FLAGS -nodefaultlibs)
    target_link_libraries(cxxrt-test-shared cxxrt-shared pthread ${CMAKE_DL_LIBS} c)
endif()

include_directories(${CMAKE_SOURCE_DIR}/src)
if(NOT DISABLE_EXCEPTIONS_RTTI)
    add_executable(cxxrt-test-foreign-exceptions test_foreign_exceptions.cc)
    set_property(TARGET cxxrt-test-foreign-exceptions PROPERTY LINK_FLAGS "-nodefaultlibs -Wl,--wrap,_Unwind_RaiseException")
    target_link_libraries(cxxrt-test-foreign-exceptions cxxrt-static gcc_s pthread ${CMAKE_DL_LIBS} c)
endif()

if(NOT NO_STATIC_TEST)
    add_test(cxxrt-test-static-test
            ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
            ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-static
            ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
            ${CMAKE_CURRENT_BINARY_DIR}/test-static-output.log)
endif()
if(NOT NO_SHARED)
    add_test(cxxrt-test-shared-test
             ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
             ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-shared
             ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
             ${CMAKE_CURRENT_BINARY_DIR}/test-shared-output.log)
endif()

if(NOT DISABLE_EXCEPTIONS_RTTI)
    add_test(cxxrt-test-foreign-exceptions
             ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-foreign-exceptions)
endif()

set(valgrind "valgrind -q")

if(TEST_VALGRIND)
    add_test(cxxrt-test-static-test-valgrind
             ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
             "${valgrind} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-static"
             ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
             ${CMAKE_CURRENT_BINARY_DIR}/test-static-output.log)
    
    add_test(cxxrt-test-shared-test-valgrind
             ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
             "${valgrind} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-shared"
             ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
             ${CMAKE_CURRENT_BINARY_DIR}/test-shared-output.log)
endif()


# Testing with libunwind
option(TEST_LIBUNWIND "Test libcxxrt with libunwind" OFF)

if(TEST_LIBUNWIND)
    if(NOT LIBUNWIND_PATH)
        message(FATAL_ERROR "Path to libunwind should be specified. Please set LIBUNWIND_PATH variable")
    endif()

    add_executable(cxxrt-test-libunwind-static ${CXXTEST_SOURCES})
    set_property(TARGET cxxrt-test-libunwind-static PROPERTY LINK_FLAGS
                 "-L${LIBUNWIND_PATH} -nodefaultlibs")
    target_link_libraries(cxxrt-test-libunwind-static cxxrt-static
                          ${LIBUNWIND_PATH}/libunwind.a pthread gcc ${CMAKE_DL_LIBS} c)

    add_test(cxxrt-test-libunwind-static-test
             ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
             ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-libunwind-static
             ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
             ${CMAKE_CURRENT_BINARY_DIR}/test-libunwind-static-output.log)

    if(NOT NO_SHARED)
        add_executable(cxxrt-test-libunwind-shared ${CXXTEST_SOURCES})
        set_property(TARGET cxxrt-test-libunwind-shared PROPERTY LINK_FLAGS
                     "-L${LIBUNWIND_PATH} -nodefaultlibs")
        target_link_libraries(cxxrt-test-libunwind-shared cxxrt-shared
                             ${LIBUNWIND_PATH}/libunwind.so  unwind pthread ${CMAKE_DL_LIBS} c)

        add_test(cxxrt-test-libunwind-shared-test
                 ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
                 ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-libunwind-shared
                 ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
                 ${CMAKE_CURRENT_BINARY_DIR}/test-libunwind-shared-output.log)
    endif()

    if(TEST_VALGRIND)
        add_test(cxxrt-test-libunwind-static-test-valgrind
                 ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
                 "${valgrind} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-libunwind-static"
                 ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
                 ${CMAKE_CURRENT_BINARY_DIR}/test-libunwind-static-output.log)
        if(NOT NO_SHARED)
            add_test(cxxrt-test-libunwind-shared-test-valgrind
                     ${CMAKE_CURRENT_SOURCE_DIR}/run_test.sh
                     "${valgrind} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/cxxrt-test-libunwind-shared"
                     ${CMAKE_CURRENT_BINARY_DIR}/expected_output.log
                     ${CMAKE_CURRENT_BINARY_DIR}/test-libunwind-shared-output.log)
        endif()
    endif()
endif()

