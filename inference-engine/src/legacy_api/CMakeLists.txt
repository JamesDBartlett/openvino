# Copyright (C) 2018-2019 Intel Corporation
# SPDX-License-Identifier: Apache-2.0
#

set(TARGET_NAME "inference_engine_legacy")

if(ENABLE_LTO)
    ie_enable_lto()
endif()

file(GLOB_RECURSE LIBRARY_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE PUBLIC_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
file(GLOB_RECURSE NN_BUILDER_LIBRARY_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/builders/*.cpp)

set(PUBLIC_HEADERS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")

# Create named folders for the sources within the .vcproj
# Empty name lists them directly under the .vcproj

source_group("src" FILES ${LIBRARY_SRC})
source_group("include" FILES ${PUBLIC_HEADERS})

# Create object library

add_library(${TARGET_NAME}_obj OBJECT
            ${LIBRARY_SRC}
            ${NN_BUILDER_LIBRARY_SRC}
            ${PUBLIC_HEADERS})

set_ie_threading_interface_for(${TARGET_NAME}_obj)

target_compile_definitions(${TARGET_NAME}_obj PRIVATE IMPLEMENT_INFERENCE_ENGINE_API)

target_include_directories(${TARGET_NAME}_obj PRIVATE ${PUBLIC_HEADERS_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src
    $<TARGET_PROPERTY:inference_engine_plugin_api,INTERFACE_INCLUDE_DIRECTORIES>
    $<TARGET_PROPERTY:ngraph::ngraph,INTERFACE_INCLUDE_DIRECTORIES>
    $<TARGET_PROPERTY:pugixml,INTERFACE_INCLUDE_DIRECTORIES>)

function(nn_builder_disable_warnings)
    disable_deprecated_warnings()
    set_source_files_properties(${NN_BUILDER_LIBRARY_SRC} PROPERTIES COMPILE_FLAGS "${ie_c_cxx_deprecated}")
endfunction()

nn_builder_disable_warnings()

# Create shared library

add_library(${TARGET_NAME} SHARED
            $<TARGET_OBJECTS:${TARGET_NAME}_obj>
            $<TARGET_OBJECTS:inference_engine_common_obj>)

set_ie_threading_interface_for(${TARGET_NAME})

target_link_libraries(${TARGET_NAME} PRIVATE ${NGRAPH_LIBRARIES} pugixml)

add_cpplint_target(${TARGET_NAME}_cpplint FOR_TARGETS ${TARGET_NAME})
add_clang_format_target(${TARGET_NAME}_clang_format FOR_TARGETS ${TARGET_NAME})

# export targets

export(TARGETS ${TARGET_NAME} NAMESPACE IE:: APPEND FILE "${CMAKE_BINARY_DIR}/targets.cmake")

# developer package

ie_developer_export_targets(${TARGET_NAME})

# install

install(TARGETS ${TARGET_NAME}
        RUNTIME DESTINATION ${IE_CPACK_LIBRARY_PATH} COMPONENT core
        ARCHIVE DESTINATION ${IE_CPACK_LIBRARY_PATH} COMPONENT core
        LIBRARY DESTINATION ${IE_CPACK_LIBRARY_PATH} COMPONENT core)
