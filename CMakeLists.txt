cmake_minimum_required(VERSION 3.21 FATAL_ERROR)

project(GeodeBindings VERSION 1.0.0)

if (NOT DEFINED GEODE_TARGET_PLATFORM)
    message(FATAL_ERROR "GEODE_TARGET_PLATFORM is not defined.")
endif()
if (NOT DEFINED GEODE_GD_VERSION)
    message(FATAL_ERROR "GEODE_GD_VERSION is not defined.")
endif()
if (NOT DEFINED GEODE_LOADER_PATH)
    message(FATAL_ERROR "GEODE_LOADER_PATH is not defined.")
endif()

option(
    BINDINGS_VERSIONED_ONLY
    "Disables inclusion of the unversioned Enums header, allowing for builds on older Geode/Geometry Dash versions."
    OFF
)

if (NOT GEODE_BINDINGS_PATH)
    set(GEODE_BINDINGS_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bindings/${GEODE_GD_VERSION})
endif()

set(SKIP_BUILDING_CODEGEN ON)


file(GLOB GEODE_BINDINGS_FILES ${GEODE_BINDINGS_PATH}/*.bro)
file(GLOB GEODE_INLINE_SOURCES ${GEODE_BINDINGS_PATH}/inline/*.cpp)


file(GLOB GEODE_GENERATED_SOURCES ${GEODE_CODEGEN_PATH}/Geode/source/*.cpp)


add_library(${PROJECT_NAME} ${GEODE_INLINE_SOURCES} ${GEODE_GENERATED_SOURCES})

set(GEODE_INCLUDE_SYSTEM_SPECIFIER SYSTEM)

target_include_directories(${PROJECT_NAME} ${GEODE_INCLUDE_SYSTEM_SPECIFIER} PUBLIC
    ${GEODE_CODEGEN_PATH}
)

if (NOT BINDINGS_VERSIONED_ONLY)
    target_include_directories(${PROJECT_NAME} ${GEODE_INCLUDE_SYSTEM_SPECIFIER} PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/bindings/include
    )
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
    UNITY_BUILD ON
    UNITY_BUILD_BATCH_SIZE 128
    CXX_VISIBILITY_PRESET hidden
)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)
target_compile_definitions(${PROJECT_NAME} PUBLIC GEODE_COMP_GD_VERSION=${GEODE_COMP_GD_VERSION})

if (WIN32)
    target_compile_definitions(${PROJECT_NAME} PUBLIC -DNOMINMAX)
    if (MSVC)
        target_compile_options(${PROJECT_NAME} PUBLIC /bigobj)
    endif()
elseif (APPLE)
    target_compile_options(${PROJECT_NAME} PUBLIC -ffunction-sections -fdata-sections)
    target_link_options(${PROJECT_NAME} PUBLIC -dead_strip)
    if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
        add_definitions(-DGLES_SILENCE_DEPRECATION)
    endif()
elseif (ANDROID)
    target_compile_options(${PROJECT_NAME} PUBLIC -ffunction-sections -fdata-sections)
    target_link_options(${PROJECT_NAME} PUBLIC -Wl,--gc-sections)
endif()
