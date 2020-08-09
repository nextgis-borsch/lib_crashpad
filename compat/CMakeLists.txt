set(COMPAT_SOURCES)

if(APPLE)
    list(APPEND COMPAT_SOURCES
        mac/AvailabilityMacros.h
        mac/kern/exc_resource.h
        mac/mach-o/loader.h
        mac/mach/mach.h
        mac/sys/resource.h
    )
else()
    list(APPEND COMPAT_SOURCES
        non_mac/mach-o/loader.h
        non_mac/mach/mach.h
        non_mac/mach/machine.h
        non_mac/mach/vm_prot.h
    )
endif()

if(LINUX OR ANDROID)
    list(APPEND COMPAT_SOURCES
        linux/signal.h
        linux/sys/mman.cc
        linux/sys/mman.h
        linux/sys/ptrace.h
        linux/sys/user.h
    )
endif()

if(ANDROID)
    list(APPEND COMPAT_SOURCES
        android/android/api-level.cc
        android/android/api-level.h
        android/dlfcn_internal.cc
        android/dlfcn_internal.h
        android/elf.h
        android/linux/elf.h
        android/linux/prctl.h
        android/linux/ptrace.h
        android/sched.h
        android/sys/epoll.cc
        android/sys/epoll.h
        android/sys/mman.cc
        android/sys/mman.h
        android/sys/syscall.h
        android/sys/user.h
    )
endif()

if(MSVC)
    list(APPEND COMPAT_SOURCES
        win/getopt.h
        win/strings.cc
        win/strings.h
        win/sys/types.h
        win/time.cc
        win/time.h
        win/winbase.h
        win/winnt.h
        win/winternl.h
    )
elseif(MINGW)
    list(APPEND COMPAT_SOURCES
        mingw/dbghelp.h
        mingw/winnt.h
    )
else()
    list(APPEND COMPAT_SOURCES
        non_win/dbghelp.h
        non_win/minwinbase.h
        non_win/timezoneapi.h
        non_win/verrsrc.h
        non_win/windows.h
        non_win/winnt.h
    )
endif()

if(NOT LINUX AND NOT ANDROID)
    list(APPEND COMPAT_SOURCES
        non_elf/elf.h
    )
endif()

set(TARGET_NAME crashpad_compat)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

if(APPLE)
    add_library(${TARGET_NAME} INTERFACE)
    set(TI_TYPE "INTERFACE")
else()
    add_library(${TARGET_NAME} STATIC ${COMPAT_SOURCES})
    set_target_properties(${TARGET_NAME} PROPERTIES LINKER_LANGUAGE CXX)
    set(TI_TYPE "PUBLIC")
    target_link_libraries(${TARGET_NAME} PRIVATE
        $<BUILD_INTERFACE:crashpad_interface>
    )
endif()

if(MSVC)
    target_include_directories(${TARGET_NAME} ${TI_TYPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/win>")
elseif(MINGW)
    target_include_directories(${TARGET_NAME} ${TI_TYPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mingw>")
else()
    target_include_directories(${TARGET_NAME} ${TI_TYPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/non_win>")
endif()

if(APPLE)
    target_include_directories(${TARGET_NAME} ${TI_TYPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/mac>")
else()
    target_include_directories(${TARGET_NAME} ${TI_TYPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/non_mac>")
endif()

if(LINUX OR ANDROID)
    target_include_directories(${TARGET_NAME} ${TI_YPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/linux>")
endif()

if(ANDROID)
    target_include_directories(${TARGET_NAME} ${TI_YPE} "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/android>")
endif()

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME compat)
set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD 14)
add_library(crashpad::compat ALIAS ${TARGET_NAME})

crashpad_install_target(${TARGET_NAME})
