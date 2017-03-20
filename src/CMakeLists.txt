
set(CXXRT_SOURCES
    dynamic_cast.cc
    exception.cc
    guard.cc
    stdexcept.cc
    typeinfo.cc
    memory.cc
    auxhelper.cc
    libelftc_dem_gnu3.c
   )

if (CHERI_PURE)
  list(APPEND CXXRT_SOURCES unwind_stubs_cheri_pure.c)
endif()

add_library(cxxrt-static STATIC ${CXXRT_SOURCES})
add_library(cxxrt-shared SHARED ${CXXRT_SOURCES})
target_link_libraries(cxxrt-shared ${CMAKE_DL_LIBS})
target_link_libraries(cxxrt-static ${CMAKE_DL_LIBS})

set_target_properties(cxxrt-static cxxrt-shared PROPERTIES
                      OUTPUT_NAME "cxxrt"
                     )



