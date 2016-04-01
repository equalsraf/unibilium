cmake_minimum_required(VERSION 2.8.7)
project(unibilium)

include(GNUInstallDirs)

set(VERSION "1.2.0")
set(LIBDIR ${CMAKE_INSTALL_FULL_LIBDIR})
set(INCDIR ${CMAKE_INSTALL_FULL_INCLUDEDIR})
configure_file(${PROJECT_SOURCE_DIR}/unibilium.pc.in ${PROJECT_BINARY_DIR}/unibilium.pc)

if(MINGW)
  # Use POSIX compatible stdio in Mingw
  add_definitions(-D__USE_MINGW_ANSI_STDIO)
endif()

if(NOT MSVC)
  add_definitions(-std=gnu99 -pedantic -Wall -Wextra -Wundef -Wshadow -Wbad-function-cast -Wcast-align -Wwrite-strings -Wstrict-prototypes -Wmissing-prototypes -Wnested-externs -Winline -Wdisabled-optimization -O2 -pipe)
  if(NOT MINGW)
    add_definitions(-fstack-protector-all -fsanitize=address -fsanitize=undefined)
  endif()
endif()

add_definitions(-DTERMINFO_DIRS="/etc/terminfo:/lib/terminfo:/usr/share/terminfo:/usr/lib/terminfo:/usr/local/share/terminfo:/usr/local/lib/terminfo")
include_directories(${PROJECT_SOURCE_DIR})

add_library(unibilium unibilium.c uninames.c uniutil.c)
set_target_properties(unibilium PROPERTIES PUBLIC_HEADER unibilium.h
	VERSION ${VERSION})

file(GLOB TOOLS_SOURCES "tools/*.c")
foreach(f ${TOOLS_SOURCES})
	get_filename_component(t ${f} NAME_WE)
	add_executable(${t} ${f})
	target_link_libraries(${t} unibilium)
endforeach()

file(GLOB TESTS_SOURCES "t/*.c")
foreach(f ${TESTS_SOURCES})
	get_filename_component(t ${f} NAME_WE)
	add_executable("test_${t}" ${f})
	target_link_libraries("test_${t}" unibilium)
endforeach()

install(TARGETS unibilium
  PUBLIC_HEADER
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
