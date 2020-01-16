set(PNGLIB_DEFINITIONS)
set(PNGLIB_MAJOR 1)
set(PNGLIB_MINOR 6)
set(PNGLIB_RELEASE 38)
set(PNGLIB_NAME libpng${PNGLIB_MAJOR}${PNGLIB_MINOR})
set(PNGLIB_VERSION ${PNGLIB_MAJOR}.${PNGLIB_MINOR}.${PNGLIB_RELEASE})

include(GNUInstallDirs)

if(UNIX AND NOT APPLE AND NOT BEOS AND NOT HAIKU)
  find_library(M_LIBRARY m)
else()
  set(M_LIBRARY "")
endif()

# PNG_HARDWARE_OPTIMIZATIONS
# set definitions and sources for arm
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm" OR
  CMAKE_SYSTEM_PROCESSOR MATCHES "^aarch64")
  set(PNG_ARM_NEON_POSSIBLE_VALUES check on off)
  set(PNG_ARM_NEON "check" CACHE STRING "Enable ARM NEON optimizations:
     check: (default) use internal checking code;
     off: disable the optimizations;
     on: turn on unconditionally.")
  set_property(CACHE PNG_ARM_NEON PROPERTY STRINGS
     ${PNG_ARM_NEON_POSSIBLE_VALUES})
  list(FIND PNG_ARM_NEON_POSSIBLE_VALUES ${PNG_ARM_NEON} index)
  if(index EQUAL -1)
    message(FATAL_ERROR
            "PNG_ARM_NEON must be one of [${PNG_ARM_NEON_POSSIBLE_VALUES}]")
  elseif(NOT ${PNG_ARM_NEON} STREQUAL "off")
    set(libpng_arm_sources
      arm/arm_init.c
      arm/filter_neon.S
      arm/filter_neon_intrinsics.c
      arm/palette_neon_intrinsics.c)

    if(${PNG_ARM_NEON} STREQUAL "on")
      list(APPEND PNGLIB_DEFINITIONS PNG_ARM_NEON_OPT=2)
    elseif(${PNG_ARM_NEON} STREQUAL "check")
      list(APPEND PNGLIB_DEFINITIONS PNG_ARM_NEON_CHECK_SUPPORTED)
    endif()
  else()
    list(APPEND PNGLIB_DEFINITIONS PNG_ARM_NEON_OPT)
  endif()
endif()
# set definitions and sources for intel
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^i?86" OR
   CMAKE_SYSTEM_PROCESSOR MATCHES "^x86_64*")
  set(PNG_INTEL_SSE_POSSIBLE_VALUES on off)
  set(PNG_INTEL_SSE "on" CACHE STRING "Enable INTEL_SSE optimizations:
     off: disable the optimizations")
  set_property(CACHE PNG_INTEL_SSE PROPERTY STRINGS
     ${PNG_INTEL_SSE_POSSIBLE_VALUES})
  list(FIND PNG_INTEL_SSE_POSSIBLE_VALUES ${PNG_INTEL_SSE} index)
  if(index EQUAL -1)
    message(FATAL_ERROR
            "PNG_INTEL_SSE must be one of [${PNG_INTEL_SSE_POSSIBLE_VALUES}]")
  elseif(NOT ${PNG_INTEL_SSE} STREQUAL "off")
    set(libpng_intel_sources
      intel/intel_init.c
      intel/filter_sse2_intrinsics.c)
    if(${PNG_INTEL_SSE} STREQUAL "on")
    list(APPEND PNGLIB_DEFINITIONS PNG_INTEL_SSE_OPT=1)
    endif()
  else()
    list(APPEND PNGLIB_DEFINITIONS PNG_INTEL_SSE_OPT=0)
  endif()
endif()

set(PNG_LIB_NAME png)

include(CheckCSourceCompiles)



# OUR SOURCES
set(libpng_public_hdrs
  png.h
  pngconf.h
)
set(libpng_private_hdrs
  pngpriv.h
  pngdebug.h
  pnginfo.h
  pngstruct.h
)
# if(AWK AND NOT ANDROID)
#   list(APPEND libpng_private_hdrs "${CMAKE_CURRENT_BINARY_DIR}/pngprefix.h")
# endif()
set(libpng_sources
  ${libpng_public_hdrs}
  ${libpng_private_hdrs}
  png.c
  pngerror.c
  pngget.c
  pngmem.c
  pngpread.c
  pngread.c
  pngrio.c
  pngrtran.c
  pngrutil.c
  pngset.c
  pngtrans.c
  pngwio.c
  pngwrite.c
  pngwtran.c
  pngwutil.c
  ${libpng_arm_sources}
  ${libpng_intel_sources}
  ${libpng_mips_sources}
  ${libpng_powerpc_sources}
)

set(pngtest_sources pngtest.c)
set(pngvalid_sources contrib/libtests/pngvalid.c)
set(pngstest_sources contrib/libtests/pngstest.c)
set(pngunknown_sources contrib/libtests/pngunknown.c)
set(pngimage_sources contrib/libtests/pngimage.c)
set(pngfix_sources contrib/tools/pngfix.c)
set(png_fix_itxt_sources contrib/tools/png-fix-itxt.c)

configure_file(${PNG_DIR}/scripts/pnglibconf.h.prebuilt ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h)

if(MSVC)
  list(APPEND PNGLIB_DEFINITIONS _CRT_SECURE_NO_DEPRECATE)
endif()

if(PNG_DEBUG)
  list(APPEND PNGLIB_DEFINITIONS PNG_DEBUG)
endif()

set(PNG_SRC)
foreach(src ${libpng_sources})
  list(APPEND PNG_SRC ${PNG_DIR}/${src})
endforeach()

add_library(png STATIC ${PNG_SRC} "${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h")
set_target_properties(png PROPERTIES OUTPUT_NAME "${PNG_LIB_NAME}" CLEAN_DIRECT_OUTPUT 1)

list(APPEND PNG_LIB_TARGETS png)
if(MSVC)
  set_target_properties(png PROPERTIES PREFIX "lib")
endif()
target_link_libraries(png PRIVATE zlib_interface)
target_include_directories(png PUBLIC 
  ${PNG_DIR}
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)

add_library(png_interface INTERFACE)
target_include_directories(png_interface INTERFACE 
  ${PNG_DIR}
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)
