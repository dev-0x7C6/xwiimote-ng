cmake_minimum_required(VERSION 3.20)

project(
  xwiimote-ng
  VERSION 3.0.0
  DESCRIPTION "xwiimote-ng library")

include(GNUInstallDirs)

add_library(xwiimote-ng SHARED lib/core.c lib/monitor.c lib/xwiimote-ng.h)

target_compile_definitions(
  xwiimote-ng PUBLIC "XWII__EXPORT=__attribute__((visibility(\"default\")))")
set_target_properties(xwiimote-ng PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(xwiimote-ng PROPERTIES SOVERSION 3)

set_target_properties(xwiimote-ng PROPERTIES PUBLIC_HEADER lib/xwiimote-ng.h)

install(
  TARGETS xwiimote-ng
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

configure_file(libxwiimote-ng.pc.in libxwiimote-ng.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/libxwiimote-ng.pc
        DESTINATION ${CMAKE_INSTALL_DATADIR}/pkgconfig)
