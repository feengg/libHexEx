 #
# Copyright 2013 Computer Graphics Group, RWTH Aachen University
# Authors: Hans-Christian Ebke <ebke@cs.rwth-aachen.de>
#          Max Lyon <lyon@cs.rwth-aachen.de>
#
# This file is part of HexEx.
# 
# HexEx is free software: you can redistribute it and/or modify it under
# the terms of the GNU General Public License as published by the Free
# Software Foundation, either version 3 of the License, or (at your
# option) any later version.
# 
# HexEx is distributed in the hope that it will be useful, but WITHOUT ANY
# WARRANTY; without even the implied warranty of MERCHANTABILITY or
# FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
# for more details.
# 
# You should have received a copy of the GNU General Public License
# along with HexEx.  If not, see <http://www.gnu.org/licenses/>.
#   

cmake_minimum_required (VERSION 2.6)

# Only set project name if not build from within another project.
if("${PROJECT_NAME}" STREQUAL "")
    project (HexEx)
endif()

list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

find_package (OpenVolumeMesh REQUIRED)

add_definitions(-DINCLUDE_TEMPLATES)

set (SOURCES "")
set (INCLUDE_DIRS "")
set (LIBRARIES "")
set (LIBRARY_DIRS "")


file(GLOB SRCS "${CMAKE_CURRENT_LIST_DIR}/src/*.cc" "${CMAKE_CURRENT_LIST_DIR}/src/*.c")


list (APPEND SOURCES ${SRCS})
list (APPEND INCLUDE_DIRS ${OPENVOLUMEMESH_INCLUDE_DIR})
list (APPEND LIBRARY_DIRS ${OPENVOLUMEMESH_LIBRARY_DIR})
list (APPEND LIBRARIES ${OPENVOLUMEMESH_LIBRARY})

include_directories (${INCLUDE_DIRS})

link_directories (${CMAKE_BINARY_DIR} ${LIBRARY_DIRS} )

add_library (HexEx SHARED ${SOURCES})
add_library (HexExStatic STATIC ${SOURCES})

target_link_libraries (HexEx ${LIBRARIES})
target_link_libraries (HexExStatic ${LIBRARIES})

#
# In order for the exact predicates to work the compiler
# must not generate x87 FPU code as this leads to the use
# of extended precision registers which lead to
# wrong results.
#
# As SSE does not have extended precision registers,
# forcing the generation of SSE code ensures that the
# exact predicates produce correct results.
#
set (HEXEX_COMPILE_FLAGS "-Wall")
set (HEXEX_CXX_COMPILE_FLAGS "")
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set (HEXEX_COMPILE_FLAGS "${HEXEX_COMPILE_FLAGS} -msse -mfpmath=sse -pedantic -Weverything" )
    set (HEXEX_CXX_COMPILE_FLAGS "-std=c++11")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set (HEXEX_COMPILE_FLAGS "${HEXEX_COMPILE_FLAGS} -msse -mfpmath=sse -pedantic -Wextra")
    set (HEXEX_CXX_COMPILE_FLAGS "-std=c++11")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
    message (WARNING
        "You are using an Intel compiler which might generate x87 FPU code "
        "that breaks the exact predicates. If you know which compiler flags "
        "ensure that the Intel compiler produces SSE code, please patch "
        "the CMakeLists.txt and inform the author <lyon@cs.rwth-aachen.de>.")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	# There is code in exactinit() that should make sure that nor
	# x87 extended internal precision is used.
else ()
    message (WARNING
        "You are using an unknown compiler which might generate x87 FPU code "
        "that breaks the exact predicates. If you know how to detect this compiler "
        "and which flags "
        "ensure that this compiler produces SSE code, please patch "
        "the CMakeLists.txt and inform the author <lyon@cs.rwth-aachen.de>.")
endif ()

set(STL_RANGE_CHECKS false CACHE BOOL "Include STL range checks in debug mode (This option is only used in debug mode.)")
# Add a flag to check stl vectors in debugging mode
if (STL_RANGE_CHECKS)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_GLIBCXX_DEBUG -D_GLIBCXX_DEBUG_PEDANTIC" )
endif()

set_target_properties (HexEx
    PROPERTIES
    COMPILE_FLAGS "${HEXEX_COMPILE_FLAGS}"
    DEFINE_SYMBOLS "-DHEXEX_EXPORT_SYMBOLS"
    )

set_target_properties (HexExStatic
    PROPERTIES
    COMPILE_FLAGS "${HEXEX_COMPILE_FLAGS}"
    DEFINE_SYMBOLS "-DHEXEX_EXPORT_SYMBOLS"
    )

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${HEXEX_CXX_COMPILE_FLAGS}")

#
# Fake successful finder run if compiling as a dependent project.
#
if(NOT "${PROJECT_NAME}" MATCHES "HexEx")
    set (HEXEX_FOUND true PARENT_SCOPE)
    set (HEXEX_LIBRARIES HexEx PARENT_SCOPE)
    set (HEXEX_LIBRARY HexEx PARENT_SCOPE)
    set (HEXEX_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/src/" PARENT_SCOPE)
    set (HEXEX_LIBRARY_DIR "${CMAKE_CURRENT_BINARY_DIR}" CACHE PATH "The directory where the HexEx libraries can be found.")
endif()

if("${PROJECT_NAME}" MATCHES "HexEx")
   add_subdirectory(demo/cmdline_tool)
   add_subdirectory(demo/minimum_example)
endif()

#set(BUILD_UNIT_TESTS false CACHE BOOL "Whether to build the unit tests.")
#if (BUILD_UNIT_TESTS)
#    enable_testing()
#    set(GTEST_DIR CACHE PATH "Source path of googletest.")
#    if (NOT GTEST_DIR)
#        message(FATAL_ERROR "GTEST_DIR unset")
#    endif()
#    add_subdirectory(${GTEST_DIR} gtest)
#    add_subdirectory(tests)
#endif()


#find_package(Doxygen)
#if (DOXYGEN_FOUND)
#    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile @ONLY)
#    add_custom_target(qex_doc
#        ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile
#        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc
#        COMMENT "Generating Doxygen documentation" VERBATIM)
#endif()
