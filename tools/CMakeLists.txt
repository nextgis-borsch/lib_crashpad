set(TARGET_NAME crashpad_tools)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

add_library(${TARGET_NAME} STATIC
    tool_support.cc
    tool_support.h
)
target_link_libraries(${TARGET_NAME} PRIVATE
    $<BUILD_INTERFACE:crashpad_interface>
)

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME tools)
set_property(TARGET ${TARGET_NAME} PROPERTY CXX_STANDARD 14)
add_library(crashpad::tools ALIAS ${TARGET_NAME})

crashpad_install_target(${TARGET_NAME})

if(CRASHPAD_BUILD_TOOLS)
    add_executable(crashpad_database_util
        crashpad_database_util.cc
    )
    target_link_libraries(crashpad_database_util PRIVATE
        crashpad_client
        crashpad_compat
        crashpad_getopt
        crashpad_tools
    )
    crashpad_install_target(crashpad_database_util)

    add_executable(crashpad_http_upload
        crashpad_http_upload.cc
    )
    target_link_libraries(crashpad_http_upload PRIVATE
        crashpad_client
        crashpad_compat
        crashpad_getopt
        crashpad_tools
        crashpad_zlib
        ${MINICHROMIUM_LIBRARIES}
    )
    crashpad_install_target(crashpad_http_upload)

    add_executable(crashpad_generate_dump
        generate_dump.cc
    )
    target_link_libraries(crashpad_generate_dump PRIVATE
        crashpad_getopt
        crashpad_minidump
        crashpad_snapshot
        crashpad_tools
        ${MINICHROMIUM_LIBRARIES}
    )
    if(APPLE)
        # FIXME: cmake 3.13 added target_link_options
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -sectcreate __TEXT __info_plist \"${CMAKE_CURRENT_SOURCE_DIR}/mac/sectaskaccess_info.plist\"")
    endif()
    crashpad_install_target(crashpad_generate_dump)

    if(APPLE)
        add_executable(run_with_crashpad
            run_with_crashpad.cc
        )
        target_link_libraries(run_with_crashpad PRIVATE
            crashpad_client
            crashpad_compat
            crashpad_tools
            crashpad_util
            ${MINICHROMIUM_LIBRARIES}
        )
        crashpad_install_target(run_with_crashpad)

        add_executable(catch_exception_tool
            mac/catch_exception_tool.cc
        )
        target_link_libraries(catch_exception_tool PRIVATE
            crashpad_compat
            crashpad_tools
            crashpad_util
            ${MINICHROMIUM_LIBRARIES}
        )
        crashpad_install_target(catch_exception_tool)

        add_executable(exception_port_tool
            mac/exception_port_tool.cc
        )
        target_link_libraries(exception_port_tool PRIVATE
            crashpad_compat
            crashpad_tools
            crashpad_util
            ${MINICHROMIUM_LIBRARIES}
        )
        # FIXME: cmake 3.13 added target_link_options
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -sectcreate __TEXT __info_plist \"${CMAKE_CURRENT_SOURCE_DIR}/mac/sectaskaccess_info.plist\"")
        crashpad_install_target(exception_port_tool)

        add_executable(on_demand_service_tool
            mac/on_demand_service_tool.mm
        )
        target_link_libraries(on_demand_service_tool PRIVATE
            -framework CoreFoundation
            -framework Foundation
            crashpad_compat
            crashpad_tools
            crashpad_util
            ${MINICHROMIUM_LIBRARIES}
        )
        crashpad_install_target(on_demand_service_tool)
    endif()
endif()
