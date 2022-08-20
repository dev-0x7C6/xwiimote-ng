cmake_minimum_required(VERSION 3.20)

project(
  xwiimote-ng
  VERSION 3.0.0
  DESCRIPTION "xwiimote-ng library")

include(GNUInstallDirs)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/modules")
set(CURSES_NEED_NCURSES TRUE)
set(CURSES_NEED_WIDE TRUE)

find_package(Curses REQUIRED)
find_package(UDev REQUIRED)

add_library(xwiimote-ng SHARED lib/core.c lib/monitor.c lib/xwiimote-ng.h)

add_executable(xwiidump-ng tools/xwiidump.c)
add_executable(xwiishow-ng tools/xwiishow.c)
target_link_libraries(xwiishow-ng ${CURSES_LIBRARIES} xwiimote-ng -lm)
target_link_libraries(xwiimote-ng udev)

target_compile_definitions(
  xwiimote-ng PUBLIC "XWII__EXPORT=__attribute__((visibility(\"default\")))")

target_include_directories(xwiishow-ng PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/lib)

set_target_properties(xwiimote-ng PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(xwiimote-ng PROPERTIES SOVERSION 3)
set_target_properties(xwiimote-ng PROPERTIES PUBLIC_HEADER lib/xwiimote-ng.h)
install(TARGETS xwiidump-ng)
install(TARGETS xwiishow-ng)

install(
  TARGETS xwiimote-ng
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

configure_file(libxwiimote-ng.pc.in libxwiimote-ng.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/libxwiimote-ng.pc
        DESTINATION ${CMAKE_INSTALL_DATADIR}/pkgconfig)
