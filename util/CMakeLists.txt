set(TARGET_NAME crashpad_util)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

add_library(${TARGET_NAME} STATIC
    file/delimited_file_reader.cc
    file/delimited_file_reader.h
    file/directory_reader.h
    file/file_io.cc
    file/file_io.h
    file/file_reader.cc
    file/file_reader.h
    file/file_seeker.cc
    file/file_seeker.h
    file/file_writer.cc
    file/file_writer.h
    file/filesystem.h
    file/output_stream_file_writer.cc
    file/output_stream_file_writer.h
    file/scoped_remove_file.cc
    file/scoped_remove_file.h
    file/string_file.cc
    file/string_file.h
    file/file_helper.cc
    file/file_helper.h
    misc/address_sanitizer.h
    misc/address_types.h
    misc/arraysize.h
    misc/as_underlying_type.h
    misc/capture_context.h
    misc/clock.h
    misc/elf_note_types.h
    misc/from_pointer_cast.h
    misc/implicit_cast.h
    misc/initialization_state.h
    misc/initialization_state_dcheck.cc
    misc/initialization_state_dcheck.h
    misc/lexing.cc
    misc/lexing.h
    misc/memory_sanitizer.h
    misc/metrics.cc
    misc/metrics.h
    misc/paths.h
    misc/pdb_structures.cc
    misc/pdb_structures.h
    misc/random_string.cc
    misc/random_string.h
    misc/range_set.cc
    misc/range_set.h
    misc/reinterpret_bytes.cc
    misc/reinterpret_bytes.h
    misc/scoped_forbid_return.cc
    misc/scoped_forbid_return.h
    misc/symbolic_constants_common.h
    misc/time.cc
    misc/time.h
    misc/tri_state.h
    misc/uuid.cc
    misc/uuid.h
    misc/zlib.cc
    misc/zlib.h
    net/http_body.cc
    net/http_body.h
    net/http_body_gzip.cc
    net/http_body_gzip.h
    net/http_headers.h
    net/http_multipart_builder.cc
    net/http_multipart_builder.h
    net/http_transport.cc
    net/http_transport.h
    net/url.cc
    net/url.h
    numeric/checked_address_range.cc
    numeric/checked_address_range.h
    numeric/checked_range.h
    numeric/checked_vm_address_range.h
    numeric/in_range_cast.h
    numeric/int128.h
    numeric/safe_assignment.h
    process/process_id.h
    process/process_memory.cc
    process/process_memory.h
    process/process_memory_native.h
    process/process_memory_range.cc
    process/process_memory_range.h
    stdlib/aligned_allocator.cc
    stdlib/aligned_allocator.h
    stdlib/map_insert.h
    stdlib/objc.h
    stdlib/string_number_conversion.cc
    stdlib/string_number_conversion.h
    stdlib/strlcpy.cc
    stdlib/strlcpy.h
    stdlib/strnlen.cc
    stdlib/strnlen.h
    stdlib/thread_safe_vector.h
    stream/base94_output_stream.cc
    stream/base94_output_stream.h
    stream/file_encoder.cc
    stream/file_encoder.h
    stream/file_output_stream.cc
    stream/file_output_stream.h
    stream/log_output_stream.cc
    stream/log_output_stream.h
    stream/output_stream_interface.h
    stream/zlib_output_stream.cc
    stream/zlib_output_stream.h
    string/split_string.cc
    string/split_string.h
    synchronization/semaphore.h
    thread/stoppable.h
    thread/thread.cc
    thread/thread.h
    thread/thread_log_messages.cc
    thread/thread_log_messages.h
    thread/worker_thread.cc
    thread/worker_thread.h
)

if(NOT WIN32)
    target_sources(${TARGET_NAME} PRIVATE
        file/directory_reader_posix.cc
        file/file_io_posix.cc
        file/filesystem_posix.cc
        misc/clock_posix.cc
        posix/close_stdio.cc
        posix/close_stdio.h
        posix/scoped_dir.cc
        posix/scoped_dir.h
        posix/scoped_mmap.cc
        posix/scoped_mmap.h
        posix/signals.cc
        posix/signals.h
        synchronization/semaphore_posix.cc
        thread/thread_posix.cc
        posix/close_multiple.cc
        posix/close_multiple.h
        posix/double_fork_and_exec.cc
        posix/double_fork_and_exec.h
        posix/drop_privileges.cc
        posix/drop_privileges.h
        posix/process_info.h
        posix/symbolic_constants_posix.cc
        posix/symbolic_constants_posix.h
    )
endif()

if(APPLE)
    target_sources(${TARGET_NAME} PRIVATE
        mac/xattr.cc
        mac/xattr.h
        mach/composite_mach_message_server.cc
        mach/composite_mach_message_server.h
        mach/exc_client_variants.cc
        mach/exc_client_variants.h
        mach/exc_server_variants.cc
        mach/exc_server_variants.h
        mach/exception_behaviors.cc
        mach/exception_behaviors.h
        mach/exception_ports.cc
        mach/exception_ports.h
        mach/mach_extensions.cc
        mach/mach_extensions.h
        mach/mach_message.cc
        mach/mach_message.h
        mach/mach_message_server.cc
        mach/mach_message_server.h
        mach/symbolic_constants_mach.cc
        mach/symbolic_constants_mach.h
        misc/capture_context_mac.S
        misc/clock_mac.cc
        misc/paths_mac.cc
        synchronization/semaphore_mac.cc
        mac/checked_mach_address_range.h
        mac/launchd.h
        mac/launchd.mm
        mac/mac_util.cc
        mac/mac_util.h
        mac/service_management.cc
        mac/service_management.h
        mac/sysctl.cc
        mac/sysctl.h
        mach/bootstrap.cc
        mach/bootstrap.h
        mach/child_port_handshake.cc
        mach/child_port_handshake.h
        mach/child_port_server.cc
        mach/child_port_server.h
        mach/child_port_types.h
        mach/exception_types.cc
        mach/exception_types.h
        mach/notify_server.cc
        mach/notify_server.h
        mach/scoped_task_suspend.cc
        mach/scoped_task_suspend.h
        mach/task_for_pid.cc
        mach/task_for_pid.h
        net/http_transport_mac.mm
        posix/process_info_mac.cc
        process/process_memory_mac.cc
        process/process_memory_mac.h
    )
endif()

if(ANDROID)
    target_sources(${TARGET_NAME} PRIVATE
        linux/initial_signal_dispositions.cc
        linux/initial_signal_dispositions.h
    )
endif()

if(LINUX OR ANDROID)
    target_sources(${TARGET_NAME} PRIVATE
        net/http_transport_socket.cc
        linux/address_types.h
        linux/auxiliary_vector.cc
        linux/auxiliary_vector.h
        linux/checked_linux_address_range.h
        linux/direct_ptrace_connection.cc
        linux/direct_ptrace_connection.h
        linux/exception_handler_client.cc
        linux/exception_handler_client.h
        linux/exception_handler_protocol.cc
        linux/exception_handler_protocol.h
        linux/exception_information.h
        linux/memory_map.cc
        linux/memory_map.h
        linux/proc_stat_reader.cc
        linux/proc_stat_reader.h
        linux/proc_task_reader.cc
        linux/proc_task_reader.h
        linux/ptrace_broker.cc
        linux/ptrace_broker.h
        linux/ptrace_client.cc
        linux/ptrace_client.h
        linux/ptrace_connection.h
        linux/ptracer.cc
        linux/ptracer.h
        linux/scoped_pr_set_dumpable.cc
        linux/scoped_pr_set_dumpable.h
        linux/scoped_pr_set_ptracer.cc
        linux/scoped_pr_set_ptracer.h
        linux/scoped_ptrace_attach.cc
        linux/scoped_ptrace_attach.h
        linux/socket.cc
        linux/socket.h
        linux/thread_info.cc
        linux/thread_info.h
        linux/traits.h
        misc/capture_context_linux.S
        misc/paths_linux.cc
        misc/time_linux.cc
        posix/process_info_linux.cc
        process/process_memory_linux.cc
        process/process_memory_linux.h
        process/process_memory_sanitized.cc
        process/process_memory_sanitized.h
    )
endif()

if(WIN32)
    target_sources(${TARGET_NAME} PRIVATE
        file/directory_reader_win.cc
        file/file_io_win.cc
        file/filesystem_win.cc
        misc/clock_win.cc
        misc/paths_win.cc
        misc/time_win.cc
        net/http_transport_win.cc
        process/process_memory_win.cc
        process/process_memory_win.h
        synchronization/semaphore_win.cc
        thread/thread_win.cc
        win/address_types.h
        win/checked_win_address_range.h
        win/command_line.cc
        win/command_line.h
        win/context_wrappers.h
        win/critical_section_with_debug_info.cc
        win/critical_section_with_debug_info.h
        win/exception_handler_server.cc
        win/exception_handler_server.h
        win/get_function.cc
        win/get_function.h
        win/get_module_information.cc
        win/get_module_information.h
        win/handle.cc
        win/handle.h
        win/initial_client_data.cc
        win/initial_client_data.h
        win/loader_lock.cc
        win/loader_lock.h
        win/module_version.cc
        win/module_version.h
        win/nt_internals.cc
        win/nt_internals.h
        win/ntstatus_logging.cc
        win/ntstatus_logging.h
        win/process_info.cc
        win/process_info.h
        win/process_structs.h
        win/registration_protocol_win.cc
        win/registration_protocol_win.h
        win/safe_terminate_process.h
        win/scoped_handle.cc
        win/scoped_handle.h
        win/scoped_local_alloc.cc
        win/scoped_local_alloc.h
        win/scoped_process_suspend.cc
        win/scoped_process_suspend.h
        win/scoped_set_event.cc
        win/scoped_set_event.h
        win/session_end_watcher.cc
        win/session_end_watcher.h
        win/termination_codes.h
        win/xp_compat.h
        misc/capture_context_win.asm
        win/safe_terminate_process.asm
    )
endif()

target_include_directories(${TARGET_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/third_party/mini_chromium>
)

# Copied from: https://github.com/qedsoftware/crashpad/blob/3583c50a6575857abcf140f6ea3b8d11390205b3/util/CMakeLists.txt#L196-L233
if(APPLE)
    set(def_relative_files "exc.defs" "mach_exc.defs" "notify.defs")
    set(input_files "")
    foreach(x ${def_relative_files})
        # CMAKE_OSX_SYSROOT may be empty (e.g. for Makefile generators),
        # in this case files will be taken from root.
        set(full_path "${CMAKE_OSX_SYSROOT}/usr/include/mach/${x}")
        if(NOT EXISTS "${full_path}")
            message(FATAL_ERROR "File not found: ${full_path}")
        endif()
        list(APPEND input_files "${full_path}")
    endforeach()
    list(APPEND input_files "${CMAKE_CURRENT_LIST_DIR}/mach/child_port.defs")

    find_package(PythonInterp 2.7 REQUIRED)

    set(output_dir "${CMAKE_CURRENT_BINARY_DIR}/mach")
    file(MAKE_DIRECTORY "${output_dir}")

    # Create generate rule for each input file. Add each generated output
    # as a source to the target.
    foreach(input ${input_files})
        get_filename_component(name_we "${input}" NAME_WE)
        set(output_files "")
        foreach(suffix "User.c" "Server.c" ".h" "Server.h")
            list(APPEND output_files "${output_dir}/${name_we}${suffix}")
        endforeach()
        add_custom_command(
            OUTPUT
            ${output_files}
            COMMAND
            "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/mach/mig.py" "${input}" ${output_files}
            DEPENDS
            "${CMAKE_CURRENT_SOURCE_DIR}/mach/mig.py" "${input}"
        )
        target_sources(${TARGET_NAME} PRIVATE ${output_files})
    endforeach()

    include_directories("${CMAKE_CURRENT_BINARY_DIR}/..")
endif()

target_include_directories(${TARGET_NAME} PRIVATE
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
	$<BUILD_INTERFACE:${ZLIB_INCLUDE_DIR}>
)

target_link_libraries(${TARGET_NAME}
    PRIVATE
        $<BUILD_INTERFACE:crashpad_interface>
    PUBLIC
        crashpad_compat
        crashpad_zlib
        ${MINICHROMIUM_LIBRARIES}
)

if(APPLE)
    target_link_libraries(${TARGET_NAME} PRIVATE
        bsm
        "-framework CoreFoundation"
        "-framework Foundation"
        "-framework IOKit"
    )
endif()

if(WIN32)
    target_link_libraries(${TARGET_NAME} PRIVATE user32 version winhttp)
    if(MSVC)
        target_compile_options(${TARGET_NAME} PRIVATE "/wd4201")
        add_definitions(/FI"memory")
    elseif(MINGW)
        target_compile_options(${TARGET_NAME} PRIVATE 
            $<$<COMPILE_LANGUAGE:CXX>:-municode>
        )
        target_compile_definitions(${TARGET_NAME} PRIVATE 
            "__STDC_VERSION__=199901L"
            $<$<COMPILE_LANGUAGE:ASM_MASM>:__MINGW32__>
        )
        #exception_handler_server.cc missing <memory> header ?
        set_source_files_properties(
            win/exception_handler_server.cc
            PROPERTIES
            COMPILE_FLAGS
            "-include memory"
        )
    endif()
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(${TARGET_NAME} PRIVATE 
            $<$<COMPILE_LANGUAGE:CXX>:-Wno-multichar>
        )
    endif()
endif()

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME util)
add_library(crashpad::util ALIAS ${TARGET_NAME})

crashpad_install_target(${TARGET_NAME})

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
	install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/file
		${CMAKE_CURRENT_SOURCE_DIR}/misc 
		${CMAKE_CURRENT_SOURCE_DIR}/win
		${CMAKE_CURRENT_SOURCE_DIR}/numeric
		DESTINATION "${INSTALL_INC_DIR}/crashpad/util"
		FILES_MATCHING PATTERN "*.h"
	)
endif()
