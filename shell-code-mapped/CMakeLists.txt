cmake_minimum_required (VERSION 2.8)
project (CMakeSC)

#version number
set (CMakeSC_VERSION_MAJOR 1)
set (CMakeSC_VERSION_MINOR 0)

#indicate the entry point for the executable
add_executable (sc_mapped sc_mapped.c)

install (TARGETS sc_mapped DESTINATION bin)
