cmake_minimum_required(VERSION 3.8)
project(sljit)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CFLAGS "${CMAKE_C_FLAGS} -O2 -Wall -Wextra -Wconversion -Wsign-compare -Werror")
set(REGEX_CFLAGS "${CFLAGS} -fshort-wchar")

add_library(sljitLir OBJECT
        sljit_src/sljitLir.c
        )
set_target_properties(
        sljitLir PROPERTIES
        COMPILE_FLAGS
        "${CFLAGS}"
)

add_library(sljitMain OBJECT
        test_src/sljitMain.c
        )
target_include_directories(sljitMain PRIVATE sljit_src)
set_target_properties(
        sljitMain PROPERTIES
        COMPILE_FLAGS
        "${CFLAGS}"
)

add_library(regexMain OBJECT
        regex_src/regexMain.c
        )
target_include_directories(regexMain PRIVATE sljit_src)
set_target_properties(
        sljitMain PROPERTIES
        COMPILE_FLAGS
        "${REGEX_CFLAGS}"
)

add_library(regexJIT OBJECT
        regex_src/regexJIT.c
        )
target_include_directories(regexJIT PRIVATE sljit_src)
set_target_properties(
        regexJIT PROPERTIES
        COMPILE_FLAGS
        "${REGEX_CFLAGS}"
)

add_executable(sljit_test
        test_src/sljitTest.c
        sljit_src/sljitLir.c
        $<TARGET_OBJECTS:sljitMain>
        )
target_include_directories(sljit_test PRIVATE sljit_src test_src)
set_target_properties(
        sljit_test PROPERTIES
        COMPILE_FLAGS
        "-DSLJIT_HAVE_CONFIG_PRE=1 ${CFLAGS} -lm -lpthread"
)

add_executable(regex_test
        $<TARGET_OBJECTS:regexMain>
        $<TARGET_OBJECTS:regexJIT>
        $<TARGET_OBJECTS:sljitLir>
        )
set_target_properties(
        regex_test PROPERTIES
        COMPILE_FLAGS
        "${CFLAGS} -lm -lpthread"
)

macro(EXAMPLE_EXEC execname)
    message(STATUS "execname is:${execname}")
    add_executable(${execname}
            doc/tutorial/${execname}.c
            $<TARGET_OBJECTS:sljitLir>
            )
    target_include_directories(${execname} PRIVATE sljit_src)
    set_target_properties(
            ${execname} PROPERTIES
            COMPILE_FLAGS
            "-lm -lpthread"
    )
endmacro(EXAMPLE_EXEC)

EXAMPLE_EXEC(first_program)
EXAMPLE_EXEC(branch)
EXAMPLE_EXEC(loop)
EXAMPLE_EXEC(array_access)
EXAMPLE_EXEC(func_call)
EXAMPLE_EXEC(struct_access)
EXAMPLE_EXEC(temp_var)
EXAMPLE_EXEC(brainfuck)
