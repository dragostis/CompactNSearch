cmake_minimum_required(VERSION 3.2)

project(CompactNSearch)

# Visual studio solution directories.
set_property(GLOBAL PROPERTY USE_FOLDERS on)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if (UNIX)
option(APPLE_OMP_AUTOADD "Add OpenMP if using AppleClang" ON)

find_package(OpenMP QUIET)
    if(NOT "${OpenMP_FOUND}" OR NOT "${OpenMP_CXX_FOUND}")
        if("${APPLE_OMP_AUTOADD}" AND "${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang" AND NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS "7")
        message(STATUS "AppleClang >= 7.0 detected, adding OpenMP. Disable with -DAPPLE_OMP_AUTOADD=OFF")

        find_program(BREW NAMES brew)
        if(BREW)
            execute_process(COMMAND ${BREW} ls libomp RESULT_VARIABLE BREW_RESULT_CODE OUTPUT_QUIET ERROR_QUIET)
            if(BREW_RESULT_CODE)
                message(STATUS "This program supports OpenMP on Mac through Brew. Please run \"brew install libomp\"")
            else()
                execute_process(COMMAND ${BREW} --prefix libomp OUTPUT_VARIABLE BREW_LIBOMP_PREFIX OUTPUT_STRIP_TRAILING_WHITESPACE)
                set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp")
                set(OpenMP_CXX_LIB_NAMES "omp")
                set(OpenMP_omp_LIBRARY "${BREW_LIBOMP_PREFIX}/lib/libomp.dylib")
                set(OpenMP_INCLUDE_DIR "${BREW_LIBOMP_PREFIX}/include")
                include_directories("${BREW_LIBOMP_PREFIX}/include")
                message(STATUS "Using Homebrew libomp from ${BREW_LIBOMP_PREFIX}")
            endif()
        else()
            message(STATUS "This program supports OpenMP on Mac through Homebrew, installing Homebrew recommmended https://brew.sh")
        endif()
        endif()
    endif()

if(OPENMP_FOUND)
        SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()
endif (UNIX)

OPTION(BUILD_AS_SHARED_LIBS "Build all the libraries as shared" OFF)
if (BUILD_AS_SHARED_LIBS)
	if(WIN32)
		add_definitions(-DCMAKE_WINDOWS_EXPORT_ALL_SYMBOLS)
		set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS YES CACHE BOOL "Export all symbols")
	endif(WIN32)	
endif (BUILD_AS_SHARED_LIBS)

SET(CMAKE_DEBUG_POSTFIX "_d")

set (HEADER_FILES 
		include/Config.h
		include/CompactNSearch.h
		include/PointSet.h
		include/DataStructures.h)

if (BUILD_AS_SHARED_LIBS)
	add_library(CompactNSearch SHARED 
	    ${HEADER_FILES}
		src/CompactNSearch.cpp
)
else()
	add_library(CompactNSearch
	    ${HEADER_FILES}
		src/CompactNSearch.cpp
)
endif ()

target_include_directories(CompactNSearch PUBLIC include)

install(FILES "include/CompactNSearch" ${HEADER_FILES}
	DESTINATION include/)

install(TARGETS CompactNSearch
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)
	
OPTION(USE_DOUBLE_PRECISION "Use double precision"	ON)
if (USE_DOUBLE_PRECISION)
	add_definitions( -DUSE_DOUBLE)	
endif (USE_DOUBLE_PRECISION)	

option(BUILD_DEMO "Build example of how to use this library."
		ON)
if(BUILD_DEMO)
	add_subdirectory(demo)
endif(BUILD_DEMO)

