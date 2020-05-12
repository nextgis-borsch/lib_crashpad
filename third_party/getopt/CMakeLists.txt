set(TARGET_NAME crashpad_getopt)
set(TARGETS ${TARGETS} ${TARGET_NAME} PARENT_SCOPE)

if(MSVC)
    add_library(${TARGET_NAME} STATIC
        getopt.cc
        getopt.h
    )
    target_include_directories(${TARGET_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    )
    target_link_libraries(${TARGET_NAME} PRIVATE
        $<BUILD_INTERFACE:crashpad_interface>
    )
else()
    add_library(${TARGET_NAME} INTERFACE)
endif()

set_property(TARGET ${TARGET_NAME} PROPERTY EXPORT_NAME getopt)
add_library(crashpad::getopt ALIAS ${TARGET_NAME})
