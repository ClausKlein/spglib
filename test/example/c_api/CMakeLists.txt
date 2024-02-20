cmake_minimum_required(VERSION 3.25...3.29)
project(example_spglib LANGUAGES C)

# XXX: NO! CK include(../../../cmake/SpglibGlobalSetup.cmake)

find_package(Spglib REQUIRED)

add_executable(example example.c)
target_link_libraries(example PRIVATE Spglib::symspg)

if (UNIX)
	add_executable(example_full example_full.c)
	target_link_libraries(example_full PRIVATE Spglib::symspg)
endif ()
