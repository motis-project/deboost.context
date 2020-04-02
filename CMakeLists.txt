project(fcontext)
cmake_minimum_required(VERSION 3.16)

set(CMAKE_MSVC_RUNTIME_LIBRARY "")
if (MSVC)
  enable_language(CXX ASM_MASM)
else()
  enable_language(CXX ASM)
endif()

if(MSVC)
  add_definitions(-D_ITERATOR_DEBUG_LEVEL=0)
  add_definitions(-D_HAS_EXCEPTIONS=0)
endif()
add_definitions(-DBOOST_CONTEXT_EXPORT=)

# OS
if (UNIX)
  if (ANDROID)
    # Android
    if (${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
      set(CPU_ARCH "arm")
      set(ASM_EXT "aapcs_elf_gas.S")
    elseif (${CMAKE_SYSTEM_PROCESSOR} MATCHES "aarch64")
      set(CPU_ARCH "arm64")
      set(ASM_EXT "aapcs_elf_gas.S")
    elseif (${CMAKE_SYSTEM_PROCESSOR} MATCHES "i686")
      set(CPU_ARCH "i386")
      set(ASM_EXT "sysv_elf_gas.S")
    elseif (${CMAKE_SYSTEM_PROCESSOR} MATCHES "x86_64")
      set(CPU_ARCH "x86_64")
      set(ASM_EXT "sysv_elf_gas.S")
    endif()
  else()
    # PC (x86/x64)
    if (CMAKE_SIZEOF_VOID_P EQUAL 8)
      set(CPU_ARCH "x86_64")
    else()
      set(CPU_ARCH "i386")
    endif()

    if (APPLE)
      set(ASM_EXT "sysv_macho_gas.S") # OSX
    else()
      set(ASM_EXT "sysv_elf_gas.S")  # Linux/Unix
    endif()
  endif()
elseif(WIN32)
  # Windows PC
  if (CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CPU_ARCH "x86_64")
  else()
    set(CPU_ARCH "i386")
  endif()
  set(ASM_EXT "ms_pe_masm.asm")
endif()

set(ASM_SOURCES
  "asm/make_${CPU_ARCH}_${ASM_EXT}"
  "asm/jump_${CPU_ARCH}_${ASM_EXT}"
  "asm/ontop_${CPU_ARCH}_${ASM_EXT}")

add_library(fcontext STATIC "source/stack.c" ${ASM_SOURCES})
target_include_directories(fcontext
    PRIVATE include/fcontext
    INTERFACE include)
