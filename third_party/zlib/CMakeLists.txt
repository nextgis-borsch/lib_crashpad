set(TARGET_NAME crashpad_zlib)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

add_library(${TARGET_NAME} INTERFACE)

if(WIN32)
	find_anyproject(ZLIB REQUIRED SHARED OFF)
else()
	find_anyproject(ZLIB REQUIRED)
endif()

target_compile_definitions(${TARGET_NAME} INTERFACE
	ZLIB_CONST
	CRASHPAD_ZLIB_SOURCE_EXTERNAL
	$<BUILD_INTERFACE:${ZLIB_COMPILE_DEFINITIONS}>
)
target_link_libraries(${TARGET_NAME} INTERFACE $<BUILD_INTERFACE:${ZLIB_LIBRARIES}>)

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME zlib)
add_library(crashpad::zlib ALIAS ${TARGET_NAME})

crashpad_install_target(${TARGET_NAME})
