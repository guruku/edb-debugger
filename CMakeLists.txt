cmake_minimum_required (VERSION 2.8)
project (edb)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif()

# if the user has not specified which version, we try to make Qt5 happen
# if they have, we prefer their preference
if (NOT QT_VERSION)
	SET(QT_VERSION "Qt5" CACHE STRING "Version of Qt to use")
	SET_PROPERTY(CACHE QT_VERSION PROPERTY STRINGS Qt5 Qt4) 
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "{PROJECT_SOURCE_DIR}/cmake/Modules/")

include("GNUInstallDirs")

find_package(Boost 1.35 REQUIRED)
find_package(PkgConfig REQUIRED)

pkg_check_modules(CAPSTONE REQUIRED capstone>=3.0.4)
include_directories(${CAPSTONE_INCLUDE_DIRS})
link_directories(${CAPSTONE_LIBRARY_DIRS})

# eventually use this to generate version.h?
set(TARGET_VERSION_MAJOR 0)
set(TARGET_VERSION_MINOR 9)
set(TARGET_VERSION_PATCH 22)
set_property(GLOBAL PROPERTY VERSION ${TARGET_VERSION_MAJOR}.${TARGET_VERSION_MINOR}.${TARGET_VERSION_PATCH})


# TODO(eteran): how low can we make this
pkg_check_modules(GRAPHVIZ libgvc>=2.38.0)
if(${GRAPHVIZ_FOUND})
	include_directories(${GRAPHVIZ_INCLUDE_DIRS})
	link_directories(${GRAPHVIZ_LIBRARY_DIRS})
	add_definitions(-DENABLE_GRAPH)
endif()

if(${QT_VERSION} STREQUAL "Qt5")
	find_package(Qt5Core)
endif()

include_directories("include")

if(UNIX)
	include_directories("include/os/unix")
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
	include_directories("include/os/unix/linux")
endif()

if((${CMAKE_SYSTEM_PROCESSOR} MATCHES "i[3456]86") OR (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64"))
	include_directories("include/arch/x86-generic")
endif()

if(CMAKE_COMPILER_IS_GNUCC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -W -Wall -pedantic -Wunused")
	set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -W -Wall -pedantic -Wunused")
endif()


add_subdirectory(src)
add_subdirectory(plugins)

install (FILES ${CMAKE_SOURCE_DIR}/edb.1 DESTINATION ${CMAKE_INSTALL_MANDIR})
install (FILES ${CMAKE_SOURCE_DIR}/edb.desktop DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/applications/)
install (FILES ${CMAKE_SOURCE_DIR}/src/images/edb.png DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pixmaps/)
