cmake_minimum_required(VERSION 3.20)

project(
  xwiimote-ng
  VERSION 3.0.0
  DESCRIPTION "xwiimote-ng library")

include(GNUInstallDirs)
include(CheckSymbolExists)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/modules")

# Subobject initialisation requires C99
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# asprintf must be included via _GNU_SOURCE
set(CMAKE_REQUIRED_DEFINITIONS -D_GNU_SOURCE)
check_symbol_exists(asprintf stdio.h HAVE_ASPRINTF)
if (NOT HAVE_ASPRINTF)
    message(
        FATAL_ERROR("xwiimote requires asprintf")
    )
endif()
unset(CMAKE_REQUIRED_DEFINITIONS)
add_definitions(-D_GNU_SOURCE)


# Array iface_to_if_table uses a gnu extension range initializer with overrides.
set(CMAKE_C_EXTENSIONS ON)
# Fix compiler specific initialiser warnings
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-override-init")
elseif (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-initializer-overrides -Wno-gnu-designator")
endif()

find_package(UDev REQUIRED)

option(BUILD_XWIIDUMB OFF "Build xwiidump-ng utility")
option(BUILD_XWIISHOW OFF "Build xwiishow-ng utility")

add_library(xwiimote-ng SHARED lib/core.c lib/monitor.c lib/xwiimote-ng.h)
target_link_libraries(xwiimote-ng udev)

set_target_properties(xwiimote-ng PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(xwiimote-ng PROPERTIES SOVERSION 3)
set_target_properties(xwiimote-ng PROPERTIES PUBLIC_HEADER lib/xwiimote-ng.h)

if (BUILD_XWIIDUMP)
    add_executable(xwiidump-ng tools/xwiidump.c)
    install(TARGETS xwiidump-ng)
endif()

if (BUILD_XWIISHOW)
    set(CURSES_NEED_NCURSES TRUE)
    set(CURSES_NEED_WIDE TRUE)
    find_package(Curses REQUIRED)
    add_executable(xwiishow-ng tools/xwiishow.c)
    target_include_directories(xwiishow-ng PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/lib)
    target_link_libraries(xwiishow-ng ${CURSES_LIBRARIES} xwiimote-ng -lm)
    install(TARGETS xwiishow-ng)
endif()

install(
  TARGETS xwiimote-ng
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

configure_file(libxwiimote-ng.pc.in libxwiimote-ng.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/libxwiimote-ng.pc
        DESTINATION ${CMAKE_INSTALL_DATADIR}/pkgconfig)
