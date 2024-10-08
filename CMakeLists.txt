#
# This code is released under the
# Apache License Version 2.0 http://www.apache.org/licenses/.
#
# This is a modified version of the CMakeFile
# for the project https://github.com/lemire/FastPFor 
# This file can be used to 
#   1) Build a library separately. Then, this library
#      will be re-used by the Python build process
#   2) Build and run unit and performance tests.
#
cmake_minimum_required(VERSION 2.8.7)
if (NOT CMAKE_BUILD_TYPE)
		message(STATUS "No build type selected, default to Release")
		set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()
MESSAGE( STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE} )

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules")
include(AppendCompilerFlags)


project(PyFastPFor CXX C)
set(PROJECT_URL "https://github.com/searchivarius/PyFastPFOR")
set(PROJECT_DESCRIPTION "Python bindings for the FastPFOR C++ library: Fast integer compression")
include(DetectCPUFeatures)
#
# Runs compiler with "-dumpversion" and parses major/minor
# version with a regex.
#
# Taken&Modified from Boost.cmake
#
function(CXX_COMPILER_DUMPVERSION _OUTPUT_VERSION)

  exec_program(${CMAKE_CXX_COMPILER}
    ARGS ${CMAKE_CXX_COMPILER_ARG1} -dumpversion
    OUTPUT_VARIABLE COMPILER_VERSION
  )

  set(${_OUTPUT_VERSION} ${COMPILER_VERSION} PARENT_SCOPE)
endfunction()

if(NOT WIN32)
    CXX_COMPILER_DUMPVERSION(CXX_COMPILER_VERSION)
endif()
MESSAGE( STATUS "CMAKE_SIZEOF_VOID_P (should be 8): " ${CMAKE_SIZEOF_VOID_P} )
if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
   MESSAGE( STATUS "Good. You appear to have a 64-bit system. " )
else()
   MESSAGE(FATAL_ERROR "Please use a 64-bit system. " )
endif()
MESSAGE( STATUS "CMAKE_CXX_COMPILER_ID: " ${CMAKE_CXX_COMPILER_ID} )
MESSAGE( STATUS "CMAKE_C_COMPILER: " ${CMAKE_C_COMPILER} )
MESSAGE( STATUS "CXX_COMPILER_VERSION: " ${CXX_COMPILER_VERSION} )
if( SUPPORT_SSE42 )
    MESSAGE( STATUS "SSE 4.2 support detected" )
else()
    MESSAGE( STATUS "SSE 4.2 support not detected" )
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # require at least gcc 4.7
    if (CXX_COMPILER_VERSION VERSION_LESS 4.7)
        message(STATUS "GCC version must be at least 4.7!")
    endif()
    # Uncomment the following lines to see how the code compiles without AVX,SSE4.2 and/or SSE2
    #set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Ofast -lm  -DNDEBUG -std=c++11 -DHAVE_CXX0X -march=x86-64")
    #set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Ofast -lm  -DNDEBUG -std=c++11 -DHAVE_CXX0X -march=core2")
    #set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Ofast -lm  -DNDEBUG -std=c++11 -DHAVE_CXX0X -msse4.2")
    set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Wcast-align -Ofast -lm  -DNDEBUG -std=c++11 -DHAVE_CXX0X -msse4.1  -mavx  -mavx2")
    set (CMAKE_CXX_FLAGS_DEBUG   "-Wall -Wcast-align -ggdb  -lm  -std=c++11 -DHAVE_CXX0X -msse4.1  -mavx  -mavx2")
    set (CMAKE_C_FLAGS_RELEASE "-Wall -Wcast-align -Ofast -lm  -DNDEBUG -std=c99  -msse4.1  -mavx  -mavx2")
    set (CMAKE_C_FLAGS_DEBUG   "-Wall -Wcast-align -ggdb  -lm  -std=c99  -msse4.1  -mavx  -mavx2")
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
    if (CXX_COMPILER_VERSION VERSION_LESS 14.0.1)
        message(STATUS "Intel version must be at least 14.0.1!")
    endif()
    set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Ofast  -DNDEBUG -std=c++11 -DHAVE_CXX0X  -msse4.1  -mavx  -mavx2")
    set (CMAKE_CXX_FLAGS_DEBUG   "-Wall -ggdb   -std=c++11 -DHAVE_CXX0X  -msse4.1  -mavx  -mavx2")
    set (CMAKE_C_FLAGS_RELEASE "-Wall -Ofast  -DNDEBUG -std=c99  -msse4.1  -mavx  -mavx2")
    set (CMAKE_C_FLAGS_DEBUG   "-Wall -ggdb   -std=c99  -msse4.1  -mavx  -mavx2")
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    if (CXX_COMPILER_VERSION VERSION_LESS 4.2.1)
        message(STATUS  "Clang version must be at least 4.2.1!" )
    endif()
    set (CMAKE_CXX_FLAGS_RELEASE "-Wall -Wcast-align -O3 -DNDEBUG -std=c++11 -DHAVE_CXX0X -msse4.1 -msse4.1 -mavx -mavx2")
    set (CMAKE_CXX_FLAGS_DEBUG   "-Wall -Wcast-align -ggdb  -std=c++11 -DHAVE_CXX0X -msse4.1 -msse4.1 -mavx -mavx2")
    set (CMAKE_C_FLAGS_RELEASE "-Wall -Wcast-align -O3 -DNDEBUG -std=c99 -msse4.1 -msse4.1 -mavx -mavx2")
    set (CMAKE_C_FLAGS_DEBUG   "-Wall -Wcast-align -ggdb  -std=c99 -msse4.1 -msse4.1 -mavx -mavx2")
elseif(WIN32)
    # TODO add support for later versions?
    if(NOT MSVC12)
         message(STATUS "On Windows, only MSVC version 12 is supported!")
    endif()
else ()
    message(FATAL_ERROR "Please, use GCC, Clang, or the Intel compiler!")
endif()

MESSAGE( STATUS "CMAKE_CXX_FLAGS_DEBUG: " ${CMAKE_CXX_FLAGS_DEBUG} )
MESSAGE( STATUS "CMAKE_CXX_FLAGS_RELEASE: " ${CMAKE_CXX_FLAGS_RELEASE} )
MESSAGE( STATUS "CMAKE_C_FLAGS_DEBUG: " ${CMAKE_C_FLAGS_DEBUG} )
MESSAGE( STATUS "CMAKE_C_FLAGS_RELEASE: " ${CMAKE_C_FLAGS_RELEASE} )

# library target
include_directories(headers)
add_library(FastPFor STATIC
    src/bitpacking.cpp
    src/bitpackingaligned.cpp
    src/bitpackingunaligned.cpp
    src/codecfactory.cpp
    src/horizontalbitpacking.cpp
    src/simdunalignedbitpacking.cpp
    src/simdbitpacking.cpp
    src/varintdecode.c
    src/streamvbyte.c
	${HEADERS}
	)

add_executable(codecs src/codecs.cpp)
target_link_libraries(codecs FastPFor)

add_executable(unit src/unit.cpp)
target_link_libraries(unit FastPFor)

# Adding a make check target
add_custom_target(check unit DEPENDS unit)
