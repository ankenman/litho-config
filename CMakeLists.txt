cmake_minimum_required(VERSION 3.14)
project(litho_config VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(nlohmann_json QUIET) # try conan first
if(NOT nlohmann_json_FOUND)
    include(FetchContent)
    FetchContent_Declare(nlohmann_json
            GIT_REPOSITORY https://github.com/nlohmann/json
            GIT_TAG v3.11.3)
    FetchContent_MakeAvailable(nlohmann_json)
endif()

add_library(litho_config STATIC
    src/config.cpp
    src/knob_list.cpp
)

target_include_directories(litho_config PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(litho_config PUBLIC
        $<BUILD_INTERFACE:nlohmann_json::nlohmann_json>
)

target_compile_features(litho_config PUBLIC cxx_std_17)

# Alias for consumers
add_library(litho::config ALIAS litho_config)

include(GNUInstallDirs)

install(TARGETS litho_config
        EXPORT litho_configTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT litho_configTargets
        FILE litho_configTargets.cmake
        NAMESPACE litho::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/litho_config
)

include(CMakePackageConfigHelpers)

configure_package_config_file(
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/litho_configConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/litho_configConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/litho_config
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/litho_configConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/litho_config
)

# Tests
enable_testing()
add_subdirectory(test)