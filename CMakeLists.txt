#   Copyright (C) 2018 by Till Hofmann

#   This program is free software; you can redistribute it and/or modify it
#   under the terms of the GNU General Public License version 2 as published
#   by the Free Software Foundation.

cmake_minimum_required (VERSION 3.1.0)

# projectname is the same as the main-executable
project(memleax)

find_package(PkgConfig)

if (NOT PKG_CONFIG_FOUND)
  message(FATAL_ERROR "Could not find PkgConfig")
endif()

add_executable(
  memleax
  addr_maps.c
  breakpoint.c
  callstack.c
  debug_file.c
  debug_line.c
  memblock.c
  memleax.c
  proc_info.c
  ptr_backtrace.c
  symtab.c
)

target_compile_options(memleax PRIVATE -g -Wall)
if(${CMAKE_SYSTEM_NAME} STREQUAL Linux)
  target_compile_options(memleax PRIVATE -DMLX_LINUX)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL FreeBSD)
  target_compile_options(memleax PRIVATE -DMLX_FREEBSD)
  target_link_libraries(memleax PRIVATE procstat util z)
else()
  message(FATAL_ERROR "Unsupported OS: ${CMAKE_SYSTEM_NAME}. Only GNU/Linux and FreeBSD are supported.")
endif()

target_compile_options(memleax PRIVATE -DMLX_AARCH64)
target_link_libraries(memleax PRIVATE unwind-aarch64)

pkg_search_module(LIBUNWIND REQUIRED libunwind-ptrace)
target_compile_options(memleax PRIVATE ${LIBUNWIND_CFLAGS})

find_library(ELF_LIBRARY       NAMES elf)

target_link_libraries(memleax PRIVATE ${LIBUNWIND_LIBRARIES} ${ELF_LIBRARY})

find_library(DW_LIBRARY       NAMES dw)
target_link_libraries(memleax PRIVATE ${DW_LIBRARY})

install(TARGETS memleax RUNTIME DESTINATION bin)
install(FILES memleax.1 DESTINATION share/man/man1/)
