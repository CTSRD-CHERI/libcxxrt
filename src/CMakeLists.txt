
set(CXXRT_SOURCES
    guard.cc
    memory.cc
    auxhelper.cc
    libelftc_dem_gnu3.c
   )
if (CHERI_PURE)
  list(APPEND CXXRT_SOURCES unwind_stubs_cheri_pure.c)
else()
  list(APPEND CXXRT_SOURCES
      stdexcept.cc
      typeinfo.cc
      dynamic_cast.cc
      exception.cc
  )
endif()

add_library(cxxrt-static STATIC ${CXXRT_SOURCES})
add_library(cxxrt-shared SHARED ${CXXRT_SOURCES})
target_link_libraries(cxxrt-shared ${CMAKE_DL_LIBS})
target_link_libraries(cxxrt-static ${CMAKE_DL_LIBS})
target_compile_features(cxxrt-shared PRIVATE cxx_static_assert)
target_compile_features(cxxrt-static PRIVATE cxx_static_assert)
if (CHERI_PURE)
  target_link_libraries(cxxrt-shared -nostdlib -lc)
endif()

set_target_properties(cxxrt-static cxxrt-shared PROPERTIES
                      OUTPUT_NAME "cxxrt"
                     )



