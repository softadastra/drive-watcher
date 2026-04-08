cmake_minimum_required(VERSION 3.20)

if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    project(drive_watcher
        VERSION 0.1.0
        DESCRIPTION "Drive filesystem watcher"
        LANGUAGES CXX
    )
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# ========================
# Options
# ========================
option(DRIVE_WATCHER_BUILD_TESTS "Build drive-watcher tests" OFF)
option(DRIVE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)

# ========================
# Dependencies
# ========================
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/.vix/vix_deps.cmake")
    include("${CMAKE_CURRENT_SOURCE_DIR}/.vix/vix_deps.cmake")
endif()

# ========================
# Sources
# ========================
file(GLOB_RECURSE DRIVE_WATCHER_SOURCES
    CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
)

# ========================
# Library
# ========================
add_library(drive_watcher STATIC ${DRIVE_WATCHER_SOURCES})
add_library(drive::watcher ALIAS drive_watcher)

target_compile_features(drive_watcher PUBLIC cxx_std_20)

target_include_directories(drive_watcher
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# ========================
# Compiler options
# ========================
if(MSVC)
    target_compile_options(drive_watcher PRIVATE /W4 /permissive-)
    if(DRIVE_WARNINGS_AS_ERRORS)
        target_compile_options(drive_watcher PRIVATE /WX)
    endif()
else()
    target_compile_options(drive_watcher PRIVATE -Wall -Wextra -Wpedantic)
    if(DRIVE_WARNINGS_AS_ERRORS)
        target_compile_options(drive_watcher PRIVATE -Werror)
    endif()
endif()

# ========================
# Target properties
# ========================
set_target_properties(drive_watcher PROPERTIES
    OUTPUT_NAME drive_watcher
    VERSION ${PROJECT_VERSION}
    SOVERSION 0
    EXPORT_NAME watcher
)

# ========================
# Install
# ========================
install(
    TARGETS drive_watcher
    EXPORT driveWatcherTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    EXPORT driveWatcherTargets
    FILE driveWatcherTargets.cmake
    NAMESPACE drive::
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_watcher"
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/drive_watcherConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/drive_watcherConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/drive_watcherConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_watcher"
)

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/drive_watcherConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/drive_watcherConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/drive_watcher"
)

# ========================
# Tests
# ========================
if(DRIVE_WATCHER_BUILD_TESTS)
    include(CTest)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests/CMakeLists.txt")
        add_subdirectory(tests)
    endif()
endif()

# ========================
# Summary
# ========================
message(STATUS "------------------------------------------------------")
message(STATUS "drive::watcher configured (${PROJECT_VERSION})")
message(STATUS "Include dir:      ${CMAKE_CURRENT_SOURCE_DIR}/include")
message(STATUS "Build tests:      ${DRIVE_WATCHER_BUILD_TESTS}")
message(STATUS "Warnings=error:   ${DRIVE_WARNINGS_AS_ERRORS}")
message(STATUS "Build type:       ${CMAKE_BUILD_TYPE}")
message(STATUS "Install prefix:   ${CMAKE_INSTALL_PREFIX}")
message(STATUS "------------------------------------------------------")
