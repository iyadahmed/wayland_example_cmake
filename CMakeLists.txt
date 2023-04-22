cmake_minimum_required(VERSION 3.16)
project(NanoGUI)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED YES)
set(CMAKE_C_EXTENSIONS OFF)

# Finding Wayland ported from https://github.com/emersion/hello-wayland/blob/f6a8203309977af03cda94765dd61367c189bea6/Makefile
find_package(PkgConfig REQUIRED)
pkg_search_module(WaylandClient REQUIRED wayland-client)

pkg_get_variable(WAYLAND_PROTOCOLS_DIR wayland-protocols pkgdatadir)
pkg_get_variable(WAYLAND_SCANNER wayland-scanner wayland_scanner)
set(XDG_SHELL_PROTOCOL ${WAYLAND_PROTOCOLS_DIR}/stable/xdg-shell/xdg-shell.xml)

set(GENERATED_HEADERS_DIR ${CMAKE_CURRENT_BINARY_DIR}/generated_headers)
add_custom_command(
        OUTPUT ${GENERATED_HEADERS_DIR}/xdg-shell-client-protocol.h
        COMMAND ${CMAKE_COMMAND} -E make_directory ${GENERATED_HEADERS_DIR}
        COMMAND ${WAYLAND_SCANNER} client-header ${XDG_SHELL_PROTOCOL} ${GENERATED_HEADERS_DIR}/xdg-shell-client-protocol.h
)
add_custom_command(
        OUTPUT xdg-shell-protocol.c
        COMMAND ${WAYLAND_SCANNER} private-code ${XDG_SHELL_PROTOCOL} xdg-shell-protocol.c
)

# TODO: CMake library target that autoamatically sets up sources, include directories, compile options and libraries for
# other targets

# Setup main executable
add_executable(wayland_example main.c xdg-shell-protocol.c ${GENERATED_HEADERS_DIR}/xdg-shell-client-protocol.h)
target_include_directories(wayland_example PRIVATE ${GENERATED_HEADERS_DIR})
target_compile_options(wayland_example PRIVATE ${WaylandClient_CFLAGS})
target_link_libraries(wayland_example PRIVATE ${WaylandClient_LIBRARIES})
