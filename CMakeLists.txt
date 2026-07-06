cmake_minimum_required(VERSION 3.16)
project(indigo_mount_mxhd C)

include(GNUInstallDirs)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(MXHD_DRIVER_VERSION "0.1")

find_package(PkgConfig QUIET)
if(PkgConfig_FOUND)
    pkg_check_modules(INDIGO QUIET IMPORTED_TARGET indigo)
    if(NOT INDIGO_FOUND)
        pkg_check_modules(INDIGO QUIET IMPORTED_TARGET libindigo)
    endif()
endif()

add_library(indigo_mount_mxhd MODULE
    src/indigo_mount_mxhd.c
    src/indigo_mount_mxhd.h
)

if(INDIGO_FOUND AND TARGET PkgConfig::INDIGO)
    target_link_libraries(indigo_mount_mxhd PRIVATE PkgConfig::INDIGO)
else()
    find_path(INDIGO_INCLUDE_DIR
        NAMES indigo/indigo_driver.h indigo/indigo_mount_driver.h
        REQUIRED
    )
    find_library(INDIGO_LIBRARY
        NAMES indigo
        REQUIRED
    )
    target_include_directories(indigo_mount_mxhd PRIVATE ${INDIGO_INCLUDE_DIR})
    target_link_libraries(indigo_mount_mxhd PRIVATE ${INDIGO_LIBRARY})
endif()

target_link_libraries(indigo_mount_mxhd PRIVATE m pthread)
set_target_properties(indigo_mount_mxhd PROPERTIES PREFIX "")

install(TARGETS indigo_mount_mxhd
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/indigo
)
