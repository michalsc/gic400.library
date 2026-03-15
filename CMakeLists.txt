cmake_minimum_required(VERSION 3.14.0)
project(gic400 VERSION 1.0.0)

include(cmake/bin2h.cmake)
include(cmake/verstring.cmake)
include(cmake/sfdc.cmake)

get_verstring(VERSTRING)
sfdc(sfd/gic400.sfd)

if(NOT TARGET devicetree)
    add_subdirectory(devicetree.resource EXCLUDE_FROM_ALL)
endif()

add_executable(gic400.library
    src/gic400_main.c
    src/gic400_distributor.c
    src/gic400_api.c
    src/gic400_end.c
)

bin_to_header(gic400.library)

target_compile_options(gic400.library PRIVATE -Os -m68040 -fomit-frame-pointer -Wall -Wextra -DDEBUG)
target_compile_definitions(gic400.library 
    PRIVATE
        LIBRARY_IDSTRING="${VERSTRING}"
        LIBRARY_NAME="gic400.library"
        LIBRARY_VERSION=${CMAKE_PROJECT_VERSION_MAJOR}
        LIBRARY_REVISION=${CMAKE_PROJECT_VERSION_MINOR}
        LIBRARY_PRIORITY=126
)
target_include_directories(gic400.library PRIVATE include)
target_link_options(gic400.library PRIVATE -ffreestanding -nostdlib -nostartfiles -s -Wl,-Map -Wl,${CMAKE_BINARY_DIR}/gic400.map -Wl,-e,_doNotExecute -Wl,-T${CMAKE_CURRENT_SOURCE_DIR}/ldscript.lds)
target_link_libraries(gic400.library devicetree)

set_target_properties(gic400.library PROPERTIES LINK_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/ldscript.lds)

# no install for devicetree.resource itself, it goes to ROM
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/sfd DESTINATION Developer)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include DESTINATION Developer)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include DESTINATION Developer)
