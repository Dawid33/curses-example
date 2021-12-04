cmake_minimum_required(VERSION 3.20)
project(tui-app)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(Curses QUIET)
include(ExternalProject)
set(DEPS ${PROJECT_BINARY_DIR}/deps)
file(MAKE_DIRECTORY ${DEPS})
# Don't repeatedly build ExternalProjects.
set_directory_properties(PROPERTIES EP_UPDATE_DISCONNECTED true)

add_executable(tui-app
    main.cpp)

# ---------------- CURSES / PDCURSES ----------------
IF(curses_FOUND)
    message("Libwebsockets c : ${CURSES_LIBRARIES}")
    target_link_libraries(tui-app PUBLIC ${CURSES_LIBRARIES})
ELSE()
    message("Will build curses from source...")
    IF(WIN32)
        # ExternalProject_Add(target_curses
        #     GIT_REPOSITORY  "https://github.com/wmcbrine/PDCurses"
        #     GIT_TAG         3.9
        #     CONFIGURE_COMMAND ""
        #     BUILD_IN_SOURCE TRUE
        #     SOURCE_SUBDIR "wincon"
        #     BUILD_COMMAND "mingw32-make.exe"
        #     INSTALL_COMMAND "copy" "pdcurses.a" "${DEPS}" COMMAND "copy" "..\\curses.h" "${DEPS}")
        # add_dependencies(tui-app target_curses)
        # add_library(libcurses STATIC IMPORTED)
        # set_property(TARGET libcurses PROPERTY IMPORTED_LOCATION ${DEPS}/pdcurses.a)
        # target_include_directories(tui-app PRIVATE ${DEPS})
        # target_link_libraries(tui-app PUBLIC legacy_stdio_definitions.lib)
    ELSE()
        ExternalProject_Add(target_curses
            GIT_REPOSITORY  "https://github.com/mirror/ncurses"
            GIT_TAG         v6.3
            CONFIGURE_COMMAND "${PROJECT_BINARY_DIR}/target_curses-prefix/src/target_curses/configure" "--prefix=${DEPS}"
            BUILD_COMMAND "${CMAKE_MAKE_PROGRAM}"
            BUILD_IN_SOURCE TRUE)
        add_dependencies(tui-app target_curses)
        add_library(libcurses STATIC IMPORTED)
        set_property(TARGET libcurses PROPERTY IMPORTED_LOCATION ${DEPS}/lib/libncurses.a)
        # Need by ncurses
        target_link_options(tui-app PUBLIC -ldl)
        target_include_directories(tui-app PRIVATE include ${DEPS}/include)
        ENDIF()
ENDIF()

if(NOT WIN32)
    target_compile_options(tui-app PRIVATE -Wall -Wextra)
ENDIF()

target_include_directories(tui-app PRIVATE include)
target_link_libraries(tui-app PRIVATE libcurses)