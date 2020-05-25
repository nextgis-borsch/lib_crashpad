################################################################################
# Copyright (C) 2020, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)
project(crashpad)

# Some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# Set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

if(NOT DEFINED PACKAGE_VENDOR)
    set(PACKAGE_VENDOR NextGIS)
endif()

if(NOT DEFINED PACKAGE_BUGREPORT)
    set(PACKAGE_BUGREPORT info@nextgis.com)
endif()

if(OSX_FRAMEWORK AND (BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS))
  message(FATAL_ERROR "Only OSX_FRAMEWORK key or any or both BUILD_SHARED_LIBS
                       and BUILD_STATIC_LIBS keys are permitted")
endif()

if(OSX_FRAMEWORK)
    set(INSTALL_BIN_DIR "bin" CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR "Library/Frameworks" CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_INC_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Headers CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_ENGINES_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Resources/engines CACHE INTERNAL "Directory for engines" FORCE)
    set(INSTALL_SHARE_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Resources/ssl CACHE INTERNAL "Directory for data" FORCE)
    set(INSTALL_PARENT_INC_DIR ${INSTALL_INC_DIR} CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_CMAKECONF_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Resources/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
    set(SKIP_INSTALL_HEADERS ON)
    set(SKIP_INSTALL_EXECUTABLES OFF)
    set(SKIP_INSTALL_FILES ON)
    set(SKIP_INSTALL_EXPORT ON)
    set(CMAKE_MACOSX_RPATH ON)
else()
    include(GNUInstallDirs)

    set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_SHARE_DIR ${CMAKE_INSTALL_DATADIR}/ssl CACHE INTERNAL "Installation directory for share data" FORCE)
    set(INSTALL_PARENT_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_CMAKECONF_DIR ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
endif()

set(PACKAGE_NAME ${PROJECT_NAME})
string(TOUPPER ${PACKAGE_NAME} PACKAGE_UPPER_NAME)

include(util)

check_version(MAJOR_VER MINOR_VER REL_VER FIX_VER)
set(VERSION ${MAJOR_VER}.${MINOR_VER}.${REL_VER})
report_version(${PROJECT_NAME} ${VERSION})

if(OSX_FRAMEWORK)
    set(FRAMEWORK_VERSION "${MAJOR_VER}")
else()
    set(INSTALL_ENGINES_DIR ${INSTALL_LIB_DIR}/engines-${MAJOR_VER}.${MINOR_VER} CACHE INTERNAL "Directory for engines" FORCE)
endif()

include(FindAnyProject)

find_anyproject(MINICHROMIUM REQUIRED)
find_anyproject(ZLIB REQUIRED)

function(crashpad_install_target)
    if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
        install(TARGETS ${ARGN} 
			EXPORT crashpad_export
            RUNTIME DESTINATION ${INSTALL_BIN_DIR}
			ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
			LIBRARY DESTINATION ${INSTALL_LIB_DIR}
			INCLUDES DESTINATION ${INSTALL_INC_DIR}
			FRAMEWORK DESTINATION ${INSTALL_LIB_DIR}
        )
    endif()
endfunction()

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(LINUX TRUE)
endif()

if(WIN32)
    enable_language(ASM_MASM)
    if(MINGW)
        if(NOT CMAKE_ASM_MASM_COMPILER OR CMAKE_ASM_MASM_COMPILER STREQUAL "ml" OR CMAKE_ASM_MASM_COMPILER STREQUAL "ml64")
            message(WARNING "No custom ASM_MASM compiler defined via 'CMAKE_ASM_MASM_COMPILER'. Trying to use UASM...")
            set(CMAKE_ASM_MASM_COMPILER "uasm")
        endif()
        if(NOT CMAKE_ASM_MASM_FLAGS)
            set(CMAKE_ASM_MASM_FLAGS "-win64 -10") #use default compatibility flags
        endif()
    endif()
else()
    enable_language(ASM)
endif()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(crashpad_interface INTERFACE)
target_include_directories(crashpad_interface INTERFACE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
)
target_compile_definitions(crashpad_interface INTERFACE
    CRASHPAD_LSS_SOURCE_EMBEDDED
)
set_property(TARGET crashpad_interface PROPERTY EXPORT_NAME interface)

set(TARGETS crashpad_interface)

if(MSVC)
    target_compile_definitions(crashpad_interface INTERFACE
        NOMINMAX
        UNICODE
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
        _HAS_EXCEPTIONS=0
        _UNICODE
    )
    string(REGEX REPLACE "/[Ww][0123]" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
    string(REGEX REPLACE "/[Ww][0123]" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    target_compile_options(crashpad_interface INTERFACE
        $<$<COMPILE_LANGUAGE:C,CXX>:/FS>
        $<$<COMPILE_LANGUAGE:C,CXX>:/W4>
        $<$<COMPILE_LANGUAGE:C,CXX>:/WX>
        $<$<COMPILE_LANGUAGE:C,CXX>:/Zi>
        $<$<COMPILE_LANGUAGE:C,CXX>:/bigobj> # Support larger number of sections in obj file.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4100> # Unreferenced formal parameter.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4127> # Conditional expression is constant.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4324> # Structure was padded due to alignment specifier.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4351> # New behavior: elements of array will be default initialized.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4577> # 'noexcept' used with no exception handling mode specified.
        $<$<COMPILE_LANGUAGE:C,CXX>:/wd4996> # 'X' was declared deprecated.
    )
elseif(MINGW)
    # redirect to wmain
    # FIXME: cmake 3.13 added target_link_options
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -municode")
endif()

add_subdirectory(compat)
add_subdirectory(minidump)
add_subdirectory(snapshot)
add_subdirectory(util)
add_subdirectory(client)

add_subdirectory(third_party/zlib)
add_subdirectory(third_party/getopt)

add_subdirectory(tools)
add_subdirectory(handler)

# Export package ===============================================================

if(REGISTER_PACKAGE)
   # Export the package for use from the build-tree
   # (this registers the build-tree with a global CMake-registry)
   export(PACKAGE ${PACKAGE_UPPER_NAME})
endif()

configure_file(cmake/PackageConfig.cmake.in
   ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake @ONLY)

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
	install(FILES
		${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake
		DESTINATION ${INSTALL_CMAKECONF_DIR} COMPONENT dev)
		
    install(EXPORT crashpad_export 
		FILE ${PACKAGE_UPPER_NAME}Targets.cmake
        DESTINATION "${INSTALL_CMAKECONF_DIR}")
	
	export(TARGETS ${TARGETS}
		FILE ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Targets.cmake)
endif()

# Archiving ====================================================================

set(CPACK_PACKAGE_NAME "${PACKAGE_NAME}")
set(CPACK_PACKAGE_VENDOR "${PACKAGE_VENDOR}")
set(CPACK_PACKAGE_VERSION "${VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${PACKAGE_NAME} Installation")
set(CPACK_PACKAGE_RELOCATABLE TRUE)
set(CPACK_ARCHIVE_COMPONENT_INSTALL ON)
set(CPACK_GENERATOR "ZIP")
set(CPACK_MONOLITHIC_INSTALL ON)
set(CPACK_STRIP_FILES TRUE)

# Get cpack zip archive name
get_cpack_filename(${VERSION} PROJECT_CPACK_FILENAME)
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_CPACK_FILENAME})

include(CPack)