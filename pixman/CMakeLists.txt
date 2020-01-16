# detect version
file (READ ${PIXMAN_DIR}/configure.ac configure_ac)
string (REGEX REPLACE ".*pixman_major], ([0-9]+).*" "\\1" PIXMAN_VERSION_MAJOR ${configure_ac})
string (REGEX REPLACE ".*pixman_minor], ([0-9]+).*" "\\1" PIXMAN_VERSION_MINOR ${configure_ac})
string (REGEX REPLACE ".*pixman_micro], ([0-9]+).*" "\\1" PIXMAN_VERSION_MICRO ${configure_ac})
set(PIXMAN_VERSION "${PIXMAN_VERSION_MAJOR}.${PIXMAN_VERSION_MINOR}.${PIXMAN_VERSION_MICRO}")

find_package(Threads)
#arch optimizations
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/arch_detect.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/arch_configure.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/config_configure.cmake)

set(PIXMAN_INCLUDE_DIRS
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/pixman>
  $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/pixman>
  ${PIXMAN_DIR}/pixman
)

if(MSVC)
	set(CMAKE_C_FLAGS "/wd4244 /wd4146 ${CMAKE_C_FLAGS}")
endif()

set(SOURCES
    pixman.c
    pixman-access.c
    pixman-access-accessors.c
    pixman-bits-image.c
    pixman-combine32.c
    pixman-combine-float.c
    pixman-conical-gradient.c
    pixman-filter.c
    pixman-x86.c
    pixman-mips.c
    pixman-arm.c
    pixman-ppc.c
    pixman-edge.c
    pixman-edge-accessors.c
    pixman-fast-path.c
    pixman-glyph.c
    pixman-general.c
    pixman-gradient-walker.c
    pixman-image.c
    pixman-implementation.c
    pixman-linear-gradient.c
    pixman-matrix.c
    pixman-noop.c
    pixman-radial-gradient.c
    pixman-region16.c
    pixman-region32.c
    pixman-solid-fill.c
    pixman-timer.c
    pixman-trap.c
    pixman-utils.c
)

set(HEADERS
    pixman.h
    pixman-accessor.h
    pixman-combine32.h
    pixman-compiler.h
    pixman-edge-imp.h
    pixman-inlines.h
    pixman-private.h
)

set(PROC_DEFS)
#proccesor_optimizations
if(ARM)
    if (ARM_NEON)
      list(APPEND PROC_DEFS USE_ARM_NEON)
        list(APPEND SOURCES pixman-arm-neon.c
                            pixman-arm-neon-asm.S
                            pixman-arm-neon-asm-bilinear.S
        )
    endif (ARM_NEON)
    if (ARM_SIMD)
      list(APPEND PROC_DEFS USE_ARM_SIMD)
        list(APPEND SOURCES pixman-arm-simd.c
                            pixman-arm-simd-asm.S
                            pixman-arm-simd-asm-scaled.S
        )
    endif (ARM_SIMD)
    if (ARM_IWMMXT)
      list(APPEND PROC_DEFS USE_ARM_IWMMXT)
        list(APPEND SOURCES "pixman-mmx.c")
    endif()
endif(ARM)   

if(MIPS)
    if (MIPS_DSPR2)
      list(APPEND PROC_DEFS USE_MIPS_DSPR2)
        list(APPEND SOURCES pixman-mips-dspr2.c
                            pixman-mips-dspr2-asm.S
                            pixman-mips-memcpy-asm.S
        )
    endif (MIPS_DSPR2)
    if (MIPS_LOONGSON_MMI)
      list(APPEND PROC_DEFS USE_LOONGSON_MMI)
        list(APPEND SOURCES "pixman-mmx.c")
    endif()
endif(MIPS)  

if(PPC)
    if (PPC_VMX)
      list(APPEND PROC_DEFS USE_VMX)
        list(APPEND SOURCES "pixman-vmx.c")
    endif (PPC_VMX)
endif(PPC)  

if(X86)
    if (X86_MMX)
      list(APPEND PROC_DEFS USE_X86_MMX)
        list(APPEND SOURCES "pixman-mmx.c")
    endif (X86_MMX)
    if (X86_SSE2)
      list(APPEND PROC_DEFS USE_SSE2)
        list(APPEND SOURCES "pixman-sse2.c")
    endif (X86_SSE2)
    if (X86_SSSE3)
      list(APPEND PROC_DEFS USE_SSSE3)
        list(APPEND SOURCES "pixman-ssse3.c")
    endif(X86_SSSE3)
endif(X86)

if(UNIX)
    set(CMAKE_C_FLAGS -fPIC)
endif(UNIX) 

set(PIXMAN_SOURCES)
foreach(source ${SOURCES})
  list(APPEND PIXMAN_SOURCES ${PIXMAN_DIR}/pixman/${source})
endforeach()

set(PIXMAN_HEADERS)
foreach(header ${HEADERS})
  list(APPEND PIXMAN_HEADERS ${PIXMAN_DIR}/pixman/${header})
endforeach()
set(DEFS HAVE_CONFIG_H PACKAGE ${PROC_DEFS})
add_library (pixman-1_core OBJECT ${PIXMAN_SOURCES} ${PIXMAN_HEADERS})
target_include_directories(pixman-1_core PUBLIC ${PIXMAN_INCLUDE_DIRS})
target_compile_definitions(pixman-1_core PRIVATE ${DEFS})

add_library(pixman STATIC $<TARGET_OBJECTS:pixman-1_core>)
# target_include_directories(pixman PUBLIC ${PIXMAN_INCLUDE_DIRS})
target_compile_definitions(pixman PRIVATE ${DEFS} )

add_library(pixman_interface INTERFACE)
target_include_directories(pixman_interface INTERFACE ${PIXMAN_INCLUDE_DIRS})
