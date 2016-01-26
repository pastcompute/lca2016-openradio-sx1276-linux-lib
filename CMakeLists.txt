#  Copyright (c) 2016 Andrew McDonnell <bugs@andrewmcdonnell.net>
#
#  This file is part of the simple sx1276 Linux library.
#
#  SentriFarm Radio Relay is free software: you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation, either version 3 of the License, or
#  (at your option) any later version.
#
#  SentriFarm Radio Relay is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with SentriFarm Radio Relay.  If not, see <http://www.gnu.org/licenses/>.

# This software uses the following third party libraries, listed here with associated licenses:
#
# * boost : Boost Software License 1.0
#
# Before adding new libraries, reread https://wiki.52north.org/bin/view/Documentation/ThirdPartyLicensesForGPL

cmake_minimum_required(VERSION 2.8.11)   # <-- target_include_directories() needs 2.8.11

project(sx1276 C CXX)

if(CMAKE_COMPILER_IS_GNUCC)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -pthread")
endif(CMAKE_COMPILER_IS_GNUCC)

set(COMMON_FLAGS "-D_XOPEN_SOURCE=600")
set(COMMON_FLAGS_DEBUG "-g")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${COMMON_FLAGS}")
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${LIBDIR}")

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${COMMON_FLAGS_DEBUG}")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${COMMON_FLAGS_DEBUG}")

find_package(PkgConfig REQUIRED)
find_package(Threads REQUIRED)

set(Boost_USE_STATIC_LIBS OFF)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost 1.49.0 COMPONENTS system chrono thread date_time)
if(Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
endif()


option(enable_maintainer "Enable maintainer CFLAGS (-Wall -Werror)" OFF)
if(enable_maintainer)
if(CMAKE_COMPILER_IS_GNUCXX)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")
endif(CMAKE_COMPILER_IS_GNUCXX)
endif(enable_maintainer)

set(UGPIO_LIBRARY "")
if(defined ${MY_DEVICE})

if(${MY_DEVICE} STREQUAL "carambola2")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_DEVICE_CARAMBOLA2 -DHAVE_MY_UGPIO")
# Note, defining HAVE_MY_UGPIO above is a hack, we should fail the build if ugpio is not installed
# Because it means we cannot control the reset line of the SX1276 from code
# Although in a test situation, that is probably OK, we can do it using /sys/class/gpio beforehand instead
find_library(UGPIO_LIBRARY NAMES ugpio)
endif()
if(${MY_DEVICE} STREQUAL "raspi")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DHAVE_DEVICE_RASPI -DHAVE_MY_UGPIO")
find_library(UGPIO_LIBRARY NAMES ugpio)
endif()

endif()


# FIXME This should probably be a lib, sort it out later
set(MY_FILES buspirate_binary.c buspirate_spi.cpp sx1276_platform.cpp misc.cpp spidev_spi.cpp sx1276.cpp spi.hpp util.hpp)
set(MY_LIBS ${CMAKE_THREAD_LIBS_INIT} ${Boost_SYSTEM_LIBRARY} ${Boost_CHRONO_LIBRARY} ${Boost_DATE_TIME_LIBRARY} ${UGPIO_LIBRARY})

add_executable(bp_sx1276_dump bp_sx1276_dump.c buspirate_binary.c )
add_executable(sx1276_dump_regs sx1276_dump_regs.cpp ${MY_FILES})
add_executable(sx1276_test1_tx sx1276_test1_tx.cpp ${MY_FILES})
add_executable(sx1276_test1_rx sx1276_test1_rx.cpp ${MY_FILES})

target_link_libraries(sx1276_test1_tx ${MY_LIBS})
target_link_libraries(sx1276_test1_rx ${MY_LIBS})
target_link_libraries(sx1276_dump_regs ${MY_LIBS})

