
set(ZLIB_DEFS)
set(ZLIB_INCLUDE_DIRS)

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)

check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(stdint.h    HAVE_STDINT_H)
check_include_file(stddef.h    HAVE_STDDEF_H)

# Check for fseeko
check_function_exists(fseeko HAVE_FSEEKO)
if(NOT HAVE_FSEEKO)
  list(APPEND ZLIB_DEFS NO_FSEEKO)
endif()

# Check for unistd.h
check_include_file(unistd.h Z_HAVE_UNISTD_H)

if(MSVC)
  list(APPEND ZLIB_DEFS _CRT_SECURE_NO_DEPRECATE _CRT_NONSTDC_NO_DEPRECATE)
endif()

if(EXISTS ${ZLIB_DIR}/zconf.h)
  message(STATUS "Renaming")
  message(STATUS "    ${CMAKE_CURRENT_SOURCE_DIR}/zconf.h")
  message(STATUS "to 'zconf.h.included' because this file is included with zlib")
  message(STATUS "but CMake generates it automatically in the build directory.")
  file(RENAME ${ZLIB_DIR}/zconf.h ${ZLIB_DIR}/zconf.h.included)
endif()

set(ZLIB_PC ${ZLIB_DIR}/zlib.pc)
configure_file(${ZLIB_DIR}/zlib.pc.cmakein ${ZLIB_PC} @ONLY)
configure_file(${ZLIB_DIR}/zconf.h.cmakein ${CMAKE_CURRENT_BINARY_DIR}/zconf.h @ONLY)
list(APPEND ZLIB_INCLUDE_DIRS ${CMAKE_CURRENT_BINARY_DIR} ${ZLIB_DIR})

set(ZLIB_PUBLIC_HDRS
  
  zlib.h
)

set(ZLIB_PRIVATE_HDRS
  crc32.h
  deflate.h
  gzguts.h
  inffast.h
  inffixed.h
  inflate.h
  inftrees.h
  trees.h
  zutil.h
)
set(ZLIB_SRCS
  adler32.c
  compress.c
  crc32.c
  deflate.c
  gzclose.c
  gzlib.c
  gzread.c
  gzwrite.c
  inflate.c
  infback.c
  inftrees.c
  inffast.c
  trees.c
  uncompr.c
  zutil.c
)

if(CMAKE_COMPILER_IS_GNUCC)
    if(ASM686)
        set(ZLIB_ASMS ${ZLIB_DIR}/contrib/asm686/match.S)
    elseif (AMD64)
        set(ZLIB_ASMS ${ZLIB_DIR}/contrib/amd64/amd64-match.S)
    endif ()

  if(ZLIB_ASMS)
    list(APPEND ZLIB_DEFS ASMV)
		set_source_files_properties(${ZLIB_ASMS} PROPERTIES LANGUAGE C COMPILE_FLAGS -DNO_UNDERLINE)
	endif()
endif()

if(MSVC)
  if(ASM686)
    enable_language(ASM_MASM)
    set(ZLIB_ASMS
      ${ZLIB_DIR}/contrib/masmx86/inffas32.asm
      ${ZLIB_DIR}/contrib/masmx86/match686.asm
    )
  elseif (AMD64)
    enable_language(ASM_MASM)
    set(ZLIB_ASMS
      ${ZLIB_DIR}/contrib/masmx64/gvmat64.asm
      ${ZLIB_DIR}/contrib/masmx64/inffasx64.asm
    )
  endif()
	if(ZLIB_ASMS)
    list(APPEND ZLIB_DEFS ASMV ASMINF)
	endif()
endif()

# parse the full version number from zlib.h and include in ZLIB_FULL_VERSION
file(READ ${ZLIB_DIR}/zlib.h _zlib_h_contents)
string(REGEX REPLACE ".*#define[ \t]+ZLIB_VERSION[ \t]+\"([-0-9A-Za-z.]+)\".*"
  "\\1" ZLIB_FULL_VERSION ${_zlib_h_contents})

if(MINGW)
  # This gets us DLL resource information when compiling on MinGW.
  if(NOT CMAKE_RC_COMPILER)
    set(CMAKE_RC_COMPILER windres.exe)
  endif()

  add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
    COMMAND ${CMAKE_RC_COMPILER}
        -D GCC_WINDRES
        -I ${ZLIB_DIR}
        -I ${CMAKE_CURRENT_BINARY_DIR}
        -o ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj
        -i ${ZLIB_DIR}/win32/zlib1.rc)
  set(ZLIB_DLL_SRCS ${CMAKE_CURRENT_BINARY_DIR}/zlib1rc.obj)
endif(MINGW)


set(ZLIB_HEADERS)
set(_ZLIB_HEARERS ${ZLIB_PUBLIC_HDRS} ${ZLIB_PRIVATE_HDRS})
foreach(header ${_ZLIB_HEARERS})
  list(APPEND ZLIB_HEADERS ${ZLIB_DIR}/${header})
endforeach()

set(ZLIB_SOURCES)
foreach(source ${ZLIB_SRCS})
  list(APPEND ZLIB_SOURCES ${ZLIB_DIR}/${source})
endforeach()

add_library(zlib STATIC ${ZLIB_SOURCES} ${ZLIB_ASMS} ${ZLIB_HEADERS})
target_include_directories(zlib PUBLIC ${ZLIB_INCLUDE_DIRS} ${CMAKE_CURRENT_BINARY_DIR}/zconf.h)
target_compile_definitions(zlib PRIVATE ${ZLIB_DEFS})

add_library(zlib_interface INTERFACE)
target_include_directories(zlib_interface INTERFACE ${ZLIB_INCLUDE_DIRS})
