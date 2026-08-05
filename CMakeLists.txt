cmake_minimum_required(VERSION 3.14)
project(litho_config VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)

# Dependency: nlohmann/json
FetchContent_Declare(nlohmann_json
    GIT_REPOSITORY https://github.com/nlohmann/json
    GIT_TAG v3.11.3)
FetchContent_MakeAvailable(nlohmann_json)

# The library
add_library(litho_config STATIC
    src/config.cpp
    src/knob_list.cpp
)

target_include_directories(litho_config PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(litho_config PUBLIC nlohmann_json::nlohmann_json)

target_compile_features(litho_config PUBLIC cxx_std_20)

# Alias for consumers
add_library(litho::config ALIAS litho_config)

# Tests
enable_testing()
add_subdirectory(test)
