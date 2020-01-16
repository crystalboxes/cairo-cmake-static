# CMAKE_TOOLCHAIN_FILE must be set before `project' is called, which
# configures the base build environment and references the toolchain file
if (APPLE)
  if (DEFINED IOS_PLATFORM)
    if (NOT "${IOS_PLATFORM}" STREQUAL "OS"
        AND NOT "${IOS_PLATFORM}" STREQUAL "SIMULATOR"
        AND NOT "${IOS_PLATFORM}" STREQUAL "SIMULATOR64")
      message(FATAL_ERROR
        "IOS_PLATFORM must be set to either OS, SIMULATOR, or SIMULATOR64")
    endif ()
    if (NOT "${CMAKE_GENERATOR}" STREQUAL "Xcode")
      message(AUTHOR_WARNING
        "You should use Xcode generator with IOS_PLATFORM enabled to get Universal builds.")
    endif ()
    if (BUILD_SHARED_LIBS)
      message(FATAL_ERROR
        "BUILD_SHARED_LIBS can not be on with IOS_PLATFORM enabled")
    endif ()
    if (BUILD_FRAMEWORK)
      message(FATAL_ERROR
        "BUILD_FRAMEWORK can not be on with IOS_PLATFORM enabled")
    endif ()

    # iOS only uses static libraries
    set(BUILD_SHARED_LIBS OFF)

    set(CMAKE_TOOLCHAIN_FILE
      ${CMAKE_SOURCE_DIR}/builds/cmake/iOS.cmake)
  endif ()
else ()
  if (DEFINED IOS_PLATFORM)
    message(FATAL_ERROR "IOS_PLATFORM is not supported on this platform")
  endif ()
endif ()

set(VERSION_MAJOR "2")
set(VERSION_MINOR "10")
set(VERSION_PATCH "0")

set(LIBRARY_VERSION "6.16.0")
set(LIBRARY_SOVERSION "6")



# Create the configuration file
if (NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h")
  if (UNIX)
    check_include_file("unistd.h" HAVE_UNISTD_H)
    check_include_file("fcntl.h" HAVE_FCNTL_H)
    check_include_file("stdint.h" HAVE_STDINT_H)

    file(READ "${FREETYPE_DIR}/builds/unix/ftconfig.in"
      FTCONFIG_H)
    if (HAVE_UNISTD_H)
      string(REGEX REPLACE
        "#undef +(HAVE_UNISTD_H)" "#define \\1 1"
        FTCONFIG_H "${FTCONFIG_H}")
    endif ()
    if (HAVE_FCNTL_H)
      string(REGEX REPLACE
        "#undef +(HAVE_FCNTL_H)" "#define \\1 1"
        FTCONFIG_H "${FTCONFIG_H}")
    endif ()
    if (HAVE_STDINT_H)
      string(REGEX REPLACE
        "#undef +(HAVE_STDINT_H)" "#define \\1 1"
        FTCONFIG_H "${FTCONFIG_H}")
    endif ()
    string(REPLACE "/undef " "#undef "
      FTCONFIG_H "${FTCONFIG_H}")
  else()
    file(READ "${FREETYPE_DIR}/include/freetype/config/ftconfig.h"
      FTCONFIG_H)
  endif ()
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftconfig.h"
    "${FTCONFIG_H}")
endif()


if (NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h")
  # Create the options file
  file(READ "${FREETYPE_DIR}/include/freetype/config/ftoption.h"
    FTOPTION_H)
  # if (ZLIB_FOUND)
    string(REGEX REPLACE
      "/\\* +(#define +FT_CONFIG_OPTION_SYSTEM_ZLIB) +\\*/" "\\1"
      FTOPTION_H "${FTOPTION_H}")
  # endif ()
  if (BZIP2_FOUND)
    string(REGEX REPLACE
      "/\\* +(#define +FT_CONFIG_OPTION_USE_BZIP2) +\\*/" "\\1"
      FTOPTION_H "${FTOPTION_H}")
  endif ()
  # if (PNG_FOUND)
    string(REGEX REPLACE
      "/\\* +(#define +FT_CONFIG_OPTION_USE_PNG) +\\*/" "\\1"
      FTOPTION_H "${FTOPTION_H}")
  # endif ()
  if (HARFBUZZ_FOUND)
    string(REGEX REPLACE
      "/\\* +(#define +FT_CONFIG_OPTION_USE_HARFBUZZ) +\\*/" "\\1"
      FTOPTION_H "${FTOPTION_H}")
  endif ()
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/include/freetype/config/ftoption.h"
    "${FTOPTION_H}")
endif()

file(GLOB PUBLIC_HEADERS "${FREETYPE_DIR}/include/ft2build.h" "${FREETYPE_DIR}/include/freetype/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "${FREETYPE_DIR}/include/freetype/config/*.h")
file(GLOB PRIVATE_HEADERS "${FREETYPE_DIR}/include/freetype/internal/*.h")


set(_BASE_SRCS
  src/autofit/autofit.c
  src/base/ftbase.c
  src/base/ftbbox.c
  src/base/ftbdf.c
  src/base/ftbitmap.c
  src/base/ftcid.c
  src/base/ftfstype.c
  src/base/ftgasp.c
  src/base/ftglyph.c
  src/base/ftgxval.c
  src/base/ftinit.c
  src/base/ftmm.c
  src/base/ftotval.c
  src/base/ftpatent.c
  src/base/ftpfr.c
  src/base/ftstroke.c
  src/base/ftsynth.c
  src/base/ftsystem.c
  src/base/fttype1.c
  src/base/ftwinfnt.c
  src/bdf/bdf.c
  src/bzip2/ftbzip2.c
  src/cache/ftcache.c
  src/cff/cff.c
  src/cid/type1cid.c
  src/gzip/ftgzip.c
  src/lzw/ftlzw.c
  src/pcf/pcf.c
  src/pfr/pfr.c
  src/psaux/psaux.c
  src/pshinter/pshinter.c
  src/psnames/psnames.c
  src/raster/raster.c
  src/sfnt/sfnt.c
  src/smooth/smooth.c
  src/truetype/truetype.c
  src/type1/type1.c
  src/type42/type42.c
  src/winfonts/winfnt.c
)

set(BASE_SRCS)
foreach(src ${_BASE_SRCS})
  list(APPEND BASE_SRCS ${FREETYPE_DIR}/${src})
endforeach()

if (WIN32)
  enable_language(RC)
  list(APPEND BASE_SRCS ${FREETYPE_DIR}/builds/windows/ftdebug.c
                        ${FREETYPE_DIR}/src/base/ftver.rc)
elseif (WINCE)
  list(APPEND BASE_SRCS ${FREETYPE_DIR}/builds/wince/ftdebug.c)
else ()
  list(APPEND BASE_SRCS ${FREETYPE_DIR}/src/base/ftdebug.c)
endif ()

add_library(freetype STATIC
  ${PUBLIC_HEADERS}
  ${PUBLIC_CONFIG_HEADERS}
  ${PRIVATE_HEADERS}
  ${BASE_SRCS}
)

set_target_properties(
  freetype PROPERTIES
    C_VISIBILITY_PRESET hidden)

target_compile_definitions(
  freetype PRIVATE FT2_BUILD_LIBRARY)

  
if (WIN32)
  target_compile_definitions(
    freetype PRIVATE _CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_WARNINGS)
  if (BUILD_SHARED_LIBS)
    target_compile_definitions(
      freetype PRIVATE DLL_EXPORT)
  endif ()
endif ()

# Pick up ftconfig.h and ftoption.h generated above, first.
target_include_directories(
  freetype
PUBLIC
  $<INSTALL_INTERFACE:include/freetype2>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  ${FREETYPE_DIR}/include
PRIVATE
  ${CMAKE_CURRENT_BINARY_DIR}/include
  ${FREETYPE_DIR}/include
)

target_link_libraries(freetype PRIVATE zlib_interface png_interface)
