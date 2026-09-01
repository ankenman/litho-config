# litho-config

[![googletest](https://github.com/ankenman/litho-config/actions/workflows/googletest.yml/badge.svg)](https://github.com/ankenman/litho-config/actions/workflows/googletest.yml)
[![clang-format](https://github.com/ankenman/litho-config/actions/workflows/clang-format.yml/badge.svg)](https://github.com/ankenman/litho-config/actions/workflows/clang-format.yml)

A C++17 knob-based configuration library for simulators and other command-line applications.

Modules register named, typed knobs at construction time. Values can be set from the command line, a plain-text config file, or a JSON file, with well-defined precedence. `--help` output is generated automatically from registered knobs.

## Quick example

```cpp
#include <litho/config/config.h>

int main(int argc, char* argv[]) {
    auto& knobs = litho::config::get_or_create("my_module");
    auto& clock_period_ps =
        knobs.add_knob<int>("clock_period_ps", "Clock period in picoseconds", 1000);
    auto& enable_debug =
        knobs.add_knob<bool>("enable_debug", "Enable debug output", false);

    litho::config::parse_command_line(argc, argv);

    std::cout << "Clock: " << clock_period_ps.get() << " ps\n";
    std::cout << "Debug: " << (enable_debug.get() ? "on" : "off") << "\n";
}
```

Run it:

```bash
./my_program --my_module.clock_period_ps 500 --my_module.enable_debug
./my_program --help
./my_program --json config.json
./my_program --config overrides.txt
```

## Features

- Type-safe knobs (`int`, `double`, `bool`, `std::string`) with defaults and descriptions.
- Per-module knob registration; each knob is looked up as `module.knob_name`.
- Command-line parsing with a `strict` flag: non-strict silently ignores unknown knobs (useful for multi-pass registration), strict warns on typos.
- Plain-text and JSON config file loading (`parse_config_file`, `parse_json_file`).
- Config dumping (`write_config_file`, `write_json_file`) for reproducibility.
- Precedence: JSON file < text config file < command-line arguments.
- Auto-generated `--help` output listing every registered knob.

## Building

Requires a C++17 compiler and CMake 3.14+. Depends on nlohmann/json.

### With FetchContent

```bash
mkdir build && cd build
cmake ..
cmake --build .
ctest
```

Dependencies are downloaded automatically.

### With Conan

```bash
conan install . --output-folder=build --build=missing
cmake --preset conan-release
cmake --build --preset conan-release
ctest --preset conan-release
```

## Integrating into your project

### FetchContent

```cmake
include(FetchContent)
FetchContent_Declare(litho_config
    GIT_REPOSITORY https://github.com/ankenman/litho-config
    GIT_TAG v0.1.0)
FetchContent_MakeAvailable(litho_config)

target_link_libraries(my_target PRIVATE litho::config)
```

### Conan

In your `conanfile.txt`:

```
[requires]
litho_config/0.1.0

[generators]
CMakeDeps
CMakeToolchain
```

In your `CMakeLists.txt`:

```cmake
find_package(litho_config REQUIRED)
target_link_libraries(my_target PRIVATE litho::config)
```

### Installed system-wide

After building with `-DCMAKE_INSTALL_PREFIX=/somewhere`, install with `cmake --install .`. Consumers use:

```cmake
find_package(litho_config REQUIRED)
target_link_libraries(my_target PRIVATE litho::config)
```

## Multi-pass parsing

Some applications register knobs in phases — for example, a simulator whose topology loader creates modules based on early knob values. The `strict` flag supports this:

```cpp
// Phase 1: main's knobs are registered.
litho::config::parse_command_line(argc, argv, /*strict=*/false);

// Phase 2: modules are constructed, registering their knobs.
loader.load(topology_file);

// Phase 3: final parse — now strict, catches typos.
litho::config::parse_command_line(argc, argv, /*strict=*/true);
```

Non-strict silently ignores unknown knobs. Strict warns on anything unrecognized — useful as a final sanity check once all knobs are registered.

## Config file format

**Plain text** (`--config file.txt`):

```
# Comments start with #
my_module.clock_period_ps = 500
my_module.enable_debug = true
another_module.buffer_size = 2048
```

**JSON** (`--json file.json`):

```json
{
    "my_module": {
        "clock_period_ps": 500,
        "enable_debug": true
    },
    "another_module": {
        "buffer_size": 2048
    }
}
```

## License

BSD-3-Clause. See LICENSE.
