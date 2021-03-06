# This is the CMakeLists.txt (cmake system build file) for the mwsd
# (MicroWave Synthesizer Display) program, written by Jeanette C.
# This is free software distributed under the terms of the GNU General Public
# License version 3. See the file COPYING for further information on the GPL.

# Set up CMake
cmake_minimum_required (VERSION 3.1)
# Include additional find modules
set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
# Enable c++11 standard
set (CMAKE_CXX_STANDARD 11)
project (mwsd C CXX) # project name and involved programming languages
# The main executable and its source files

add_executable (mwsd main.cpp synth_info.cpp curses_mw_miner.cpp curses_mw_ui.cpp)

# Include current dire and binary
set (CMAKE_INCLUDE_CURRENT_DIR ON)
set (INCS ${INCS} ${PROJECT_BINARY_DIR})

# Find libncurses or libncursesw
set(CURSES_NEED_NCURSES TRUE)
find_package (Curses REQUIRED)
if (CURSES_FOUND AND CURSES_HAVE_NCURSES_H)
	message(STATUS "Found libncurses or libncursesw")
	set (INCS ${INCS} ${CURSES_INCLUDE_DIRS})
	set (LIBS ${LIBS} ${CURSES_LIBRARIES})
else (CURSES_FOUND AND CURSES_HAVE_NCURSES_H)
	message(FATAL_ERROR "libncurses or libncursesw required but not found")
endif (CURSES_FOUND AND CURSES_HAVE_NCURSES_H)

# find libboost_program_options and libboost_filesystem
set(Boost_USE_MULTITHREADED ON) # use multithreaded versions
if (CMAKE_SYSTEM_NAME MATCHES "Darwin") # if this is Mac OS
	set(Boost_USE_STATIC_LIBS ON) # for MAC OS
	set(Boost_USE_STATIC_RUNTIME ON) # for MAC OS
endif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
find_package(Boost 1.58 REQUIRED COMPONENTS program_options filesystem date_time system)
if (Boost_PROGRAM_OPTIONS_FOUND)
	message(STATUS "found libboost_program_options")
	set (INCS ${INCS} ${Boost_INCLUDE_DIRS})
	set (LIBS ${LIBS} ${Boost_PROGRAM_OPTIONS_LIBRARY} ${Boost_SYSTEM_LIBRARY})
else (Boost_PROGRAM_OPTIONS_FOUND)
	message(FATAL_ERROR "libboost_program_options is required but not found")
endif (Boost_PROGRAM_OPTIONS_FOUND)

if (Boost_FILESYSTEM_FOUND)
	message(STATUS "Found libboost_filesystem")
	set(LIBS ${LIBS} ${Boost_FILESYSTEM_LIBRARY})
else (Boost_FILESYSTEM_FOUND)
	message(FATAL_ERROR "libboost_filesystem is required but not found")
endif (Boost_FILESYSTEM_FOUND)

if (Boost_DATE_TIME_FOUND)
	message(STATUS "Found libboost_date_time")
	set(LIBS ${LIBS} ${Boost_DATE_TIME_LIBRARY})
else (Boost_DATE_TIME_FOUND)
	message(FATAL_ERROR "libboost_date_time is required but not found")
endif (Boost_DATE_TIME_FOUND)

# find RtMidi
find_package (RTMIDI REQUIRED)
if (RTMIDI_FOUND)
	message(STATUS "Found librtmidi")
	set (LIBS ${LIBS} ${RTMIDI_LIBRARIES})
	set (INCS ${INCS} ${RTMIDI_INCLUDE_DIRS})
	set (DEFS ${DEFS} ${RTMIDI_DEFINITIONS})
else (RTMIDI_FOUND)
	message(FATAL_ERROR "librtmidi not found, but required")
endif(RTMIDI_FOUND)

# Find threads library
set (CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package (Threads REQUIRED)
if (Threads_FOUND)
	message(STATUS "Found threads library")
	set (LIBS ${LIBS} ${CMAKE_THREAD_LIBS_INIT})
else (Threads_FOUND)
	message(FATAL_ERROR "threads not found, but required.")
endif (Threads_FOUND)

include_directories (${INCS})
target_link_libraries (mwsd ${LIBS})

install (TARGETS mwsd DESTINATION bin)
install (FILES mwsd.1 DESTINATION man/man1)
