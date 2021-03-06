cmake_minimum_required(VERSION 3.1)
project(aws-c-mqtt C)

if (DEFINED CMAKE_PREFIX_PATH)
    file(TO_CMAKE_PATH ${CMAKE_PREFIX_PATH} CMAKE_PREFIX_PATH)
endif()

if (DEFINED CMAKE_INSTALL_PREFIX)
    file(TO_CMAKE_PATH ${CMAKE_INSTALL_PREFIX} CMAKE_INSTALL_PREFIX)
endif()

if (UNIX AND NOT APPLE)
    include(GNUInstallDirs)
elseif(NOT DEFINED CMAKE_INSTALL_LIBDIR)
    set(CMAKE_INSTALL_LIBDIR "lib")
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_PREFIX_PATH}/${CMAKE_INSTALL_LIBDIR}/cmake")

include(AwsCFlags)
include(AwsSharedLibSetup)
include(AwsSanitizers)
include(CheckCCompilerFlag)

file(GLOB AWS_MQTT_HEADERS
        "include/aws/mqtt/*.h"
        )

file(GLOB AWS_MQTT_PRIV_HEADERS
        "include/aws/mqtt/private/*.h"
        )

file(GLOB AWS_MQTT_SRC
        "source/*.c"
        )

file(GLOB MQTT_HEADERS
        ${AWS_MQTT_HEADERS}
        ${AWS_MQTT_PRIV_HEADERS}
        )

file(GLOB MQTT_SRC
        ${AWS_MQTT_SRC}
        )

add_library(${CMAKE_PROJECT_NAME} ${MQTT_HEADERS} ${MQTT_SRC})
aws_set_common_properties(${CMAKE_PROJECT_NAME})
aws_prepare_symbol_visibility_args(${CMAKE_PROJECT_NAME} "AWS_MQTT")

aws_add_sanitizers(${CMAKE_PROJECT_NAME})

if (BUILD_SHARED_LIBS)
    target_compile_definitions(${CMAKE_PROJECT_NAME} PUBLIC "-DAWS_MQTT_USE_IMPORT_EXPORT")
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE "-DAWS_MQTT_EXPORTS")
endif()

# We are not ABI stable yet
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES VERSION 1.0.0)
set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES SOVERSION 0unstable)

target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

find_package(aws-c-io REQUIRED)
target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC AWS::aws-c-io)

aws_prepare_shared_lib_exports(${CMAKE_PROJECT_NAME})

install(FILES ${AWS_MQTT_HEADERS} DESTINATION "include/aws/mqtt" COMPONENT Development)
install(FILES ${AWS_MQTT_TESTING_HEADERS} DESTINATION "include/aws/testing/mqtt" COMPONENT Development)


install(EXPORT "${CMAKE_PROJECT_NAME}-targets"
        DESTINATION "${LIBRARY_DIRECTORY}/${CMAKE_PROJECT_NAME}/cmake/"
        NAMESPACE AWS::
        COMPONENT Development)

configure_file("cmake/${CMAKE_PROJECT_NAME}-config.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
        @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
        DESTINATION "${LIBRARY_DIRECTORY}/${CMAKE_PROJECT_NAME}/cmake/"
        COMPONENT Development)

include(CTest)
if (BUILD_TESTING)
    add_subdirectory(tests)
endif ()

