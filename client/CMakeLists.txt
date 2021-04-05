set(TARGET_NAME crashpad_client)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

add_library(${TARGET_NAME} STATIC
    annotation.cc
    annotation.h
    annotation_list.cc
    annotation_list.h
    crash_report_database.cc
    crash_report_database.h
    crashpad_client.h
    crashpad_info.cc
    crashpad_info.h
    prune_crash_reports.cc
    prune_crash_reports.h
    settings.cc
    settings.h
    simple_address_range_bag.h
    simple_string_dictionary.h
    simulate_crash.h
)

if(APPLE)
    target_sources(${TARGET_NAME} PRIVATE
        crash_report_database_mac.mm
        crashpad_client_mac.cc
        simulate_crash_mac.cc
        simulate_crash_mac.h
    )
endif()

if(LINUX OR ANDROID)
    target_sources(${TARGET_NAME} PRIVATE
        crashpad_client_linux.cc
        simulate_crash_linux.h
        client_argv_handling.cc
        client_argv_handling.h
        crashpad_info_note.S
        crash_report_database_generic.cc
    )
endif()

if(WIN32)
    target_sources(${TARGET_NAME} PRIVATE
        crash_report_database_win.cc
        crashpad_client_win.cc
        simulate_crash_win.h
    )
endif()

target_include_directories(${TARGET_NAME} INTERFACE
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>"
    "$<INSTALL_INTERFACE:include/crashpad/>"
)

target_include_directories(${TARGET_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/third_party/mini_chromium>
)

target_link_libraries(${TARGET_NAME}
	PRIVATE
		$<BUILD_INTERFACE:crashpad_interface>
    PUBLIC
        crashpad_compat
        crashpad_util
        ${MINICHROMIUM_LIBRARIES}
)
target_compile_features(${TARGET_NAME} PUBLIC cxx_std_14)

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME crashpad)
add_library(crashpad::client ALIAS ${TARGET_NAME})

if(WIN32)
    target_link_libraries(${TARGET_NAME} PRIVATE rpcrt4)
    if(MSVC)
        target_compile_options(${TARGET_NAME} PRIVATE "/wd4201")
    elseif(MINGW)
        target_compile_options(${TARGET_NAME} PUBLIC
            "-municode"
        )
    endif()
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(${TARGET_NAME} PRIVATE 
            "-Wno-multichar"
            "-Wno-attributes"
        )
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(${TARGET_NAME} PRIVATE 
            "-Wno-unknown-attributes"
            "-Wno-unknown-pragmas"
        )
    endif()
endif()

crashpad_install_target(${TARGET_NAME})

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
	install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/"
		DESTINATION "${INSTALL_INC_DIR}/crashpad/client"
		FILES_MATCHING PATTERN "*.h"
	)
endif()
