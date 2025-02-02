cmake_minimum_required(VERSION 3.20...3.29)
project(example_spglib LANGUAGES C)


find_package(PkgConfig REQUIRED)
pkg_check_modules(spglib REQUIRED IMPORTED_TARGET spglib)

add_executable(main_c src/main.c)
target_link_libraries(main_c PRIVATE PkgConfig::spglib)

enable_testing()

add_test(NAME smoke_test_c
		COMMAND $<TARGET_FILE:main_c>
)
set_tests_properties(smoke_test_c PROPERTIES
		PASS_REGULAR_EXPRESSION "^Spglib version: ${spglib_VERSION}\n*$"
)

if (SPGLIB_WITH_Fortran)
	enable_language(Fortran)
	pkg_check_modules(spglib_fortran REQUIRED IMPORTED_TARGET spglib_f08)
	add_executable(main_Fortran src/main.F90)
	target_link_libraries(main_Fortran PRIVATE PkgConfig::spglib_fortran)
	add_test(NAME smoke_test_Fortran
			COMMAND $<TARGET_FILE:main_Fortran>
	)
	set_tests_properties(smoke_test_Fortran PROPERTIES
			PASS_REGULAR_EXPRESSION
			"^[ \n]*Spglib version: ${spglib_VERSION}[ \n]*Spglib_Fortran version: ${spglib_fortran_VERSION}[ \n]*$"
	)
endif ()
