cmake_minimum_required(VERSION 3.20)

project(
  xwiimote-ng
  VERSION 3.0.0
  DESCRIPTION "xwiimote-ng library")

include(GNUInstallDirs)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/modules")

find_package(UDev REQUIRED)

option(BUILD_XWIIDUMB OFF "Build xwiidump-ng utility")
option(BUILD_XWIISHOW OFF "Build xwiishow-ng utility")

add_library(xwiimote-ng SHARED lib/core.c lib/monitor.c lib/xwiimote-ng.h)
target_link_libraries(xwiimote-ng udev)

set_target_properties(xwiimote-ng PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(xwiimote-ng PROPERTIES SOVERSION 3)
set_target_properties(xwiimote-ng PROPERTIES PUBLIC_HEADER lib/xwiimote-ng.h)

if (BUILD_XWIIDUMB)
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
