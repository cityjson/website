---
layout: default
title: C++
parent: FlatCityBuf
nav_order: 6
has_children: false
permalink: /flatcitybuf/cpp/
---

# Using FlatCityBuf with C++

## Table of contents

{: .no_toc .text-delta }

1. TOC
   {:toc}

---

C++ bindings allow you to read, query, and write FCB files from native C++ applications. The bindings are built on top of the Rust core via [CXX](https://cxx.rs/) and require **C++17**.

<i class="fab fa-github"></i> Full documentation, build-from-source instructions, and examples: [src/cpp on GitHub](https://github.com/cityjson/flatcitybuf/tree/main/src/cpp)

## Installation

Pre-built static libraries are available for Linux, macOS, and Windows on each [GitHub Release](https://github.com/cityjson/flatcitybuf/releases).

{% raw %}

```bash
# Download (example for Linux x86_64)
curl -LO https://github.com/cityjson/flatcitybuf/releases/latest/download/fcb_cpp-linux-x86_64.tar.gz

# Extract
mkdir -p fcb_cpp
tar -xzf fcb_cpp-linux-x86_64.tar.gz -C fcb_cpp
```

{% endraw %}

Each package contains:

```text
├── libfcb_cpp.a   # Static library (Rust-compiled core)
├── lib.rs.h       # CXX bridge generated header
├── lib.rs.cc      # CXX bridge generated source (compile with your code)
└── fcb.h          # High-level API header
```

See the full [INSTALL guide](https://github.com/cityjson/flatcitybuf/blob/main/src/cpp/INSTALL.md) for macOS/Windows instructions, system-wide installation, and Makefile integration.

## CMake integration

A minimal `CMakeLists.txt` to get started:

{% raw %}

```cmake
cmake_minimum_required(VERSION 3.16)
project(my_app LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)

set(FCB_DIR ${CMAKE_SOURCE_DIR}/fcb_cpp)

add_executable(my_app main.cpp ${FCB_DIR}/lib.rs.cc)
target_include_directories(my_app PRIVATE ${FCB_DIR})
target_link_libraries(my_app ${FCB_DIR}/libfcb_cpp.a)

# Platform-specific dependencies
if(APPLE)
    target_link_libraries(my_app
        "-framework Security"
        "-framework CoreFoundation"
        "-framework SystemConfiguration"
    )
elseif(UNIX)
    target_link_libraries(my_app pthread dl m)
endif()
```

{% endraw %}

## Reading an FCB file and metadata

{% raw %}

```cpp
#include "lib.rs.h"
#include <iostream>

int main(int argc, char* argv[]) {
    try {
        auto reader = fcb::fcb_reader_open(argv[1]);

        // Inspect metadata
        auto meta = fcb::fcb_reader_metadata(*reader);
        std::cout << "Features: " << meta.features_count << std::endl;
        std::cout << "Spatial index: " << (meta.has_spatial_index ? "yes" : "no") << std::endl;

        // Iterate all features
        auto iter = fcb::fcb_reader_select_all(std::move(reader));
        while (fcb::fcb_iterator_next(*iter)) {
            auto feature = fcb::fcb_iterator_current(*iter);
            std::cout << "ID: " << std::string(feature.id) << std::endl;
            // feature.json contains a CityJSONFeature JSON string
        }
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }
    return 0;
}
```

{% endraw %}

## Spatial query with bounding box

Use the packed R-tree index for efficient spatial filtering:

{% raw %}

```cpp
auto reader = fcb::fcb_reader_open("buildings.fcb");

// Coordinates must be in the same CRS as the FCB file
fcb::BoundingBox bbox;
bbox.min_x = 85000.0;
bbox.min_y = 446000.0;
bbox.max_x = 85100.0;
bbox.max_y = 446100.0;

auto iter = fcb::fcb_reader_select_bbox(std::move(reader), bbox);

while (fcb::fcb_iterator_next(*iter)) {
    auto feature = fcb::fcb_iterator_current(*iter);
    std::cout << "Feature in bbox: " << std::string(feature.id) << std::endl;
}
```

{% endraw %}

## Writing an FCB file

You can create FCB files from CityJSON metadata and features:

{% raw %}

```cpp
#include "lib.rs.h"

int main() {
    // 1. Prepare CityJSON metadata
    std::string metadata = R"({
        "type": "CityJSON",
        "version": "2.0",
        "transform": {
            "scale": [0.001, 0.001, 0.001],
            "translate": [0.0, 0.0, 0.0]
        },
        "metadata": {
            "referenceSystem": "https://www.opengis.net/def/crs/EPSG/0/7415"
        }
    })";

    auto writer = fcb::fcb_writer_new(metadata);

    // 2. Add CityJSONFeature objects
    std::string feature = R"({
        "type": "CityJSONFeature",
        "id": "building_1",
        "CityObjects": {
            "building_1": {
                "type": "Building",
                "attributes": {"yearOfConstruction": 2005},
                "geometry": []
            }
        },
        "vertices": []
    })";
    fcb::fcb_writer_add_feature(*writer, feature);

    // 3. Write to disk (consumes the writer)
    fcb::fcb_writer_write(std::move(writer), "output.fcb");

    return 0;
}
```

{% endraw %}

## Converting CityJSONSeq to FCB

A common workflow is to convert a CityJSONSeq (`.city.jsonl`) file to FCB:

{% raw %}

```cpp
#include "lib.rs.h"
#include <fstream>
#include <iostream>

int main() {
    std::ifstream infile("input.city.jsonl");
    std::string header_line;
    std::getline(infile, header_line);  // First line is CityJSON metadata

    auto writer = fcb::fcb_writer_new(header_line);

    // Remaining lines are CityJSONFeature objects
    std::string line;
    while (std::getline(infile, line)) {
        if (!line.empty()) {
            fcb::fcb_writer_add_feature(*writer, line);
        }
    }

    fcb::fcb_writer_write(std::move(writer), "output.fcb");

    // Verify by reading it back
    auto reader = fcb::fcb_reader_open("output.fcb");
    auto meta = fcb::fcb_reader_metadata(*reader);
    std::cout << "Written " << meta.features_count << " features" << std::endl;

    return 0;
}
```

{% endraw %}

## Limitations

- **HTTP / remote reading** is not available through C++ bindings. Use the CLI (`fcb info -i <url>`) or download files first.
- Features are returned as **JSON strings** — use a JSON library like [nlohmann/json](https://github.com/nlohmann/json) to parse them.

## Further resources

- [README — full API reference & build-from-source](https://github.com/cityjson/flatcitybuf/tree/main/src/cpp)
- [INSTALL — pre-built binaries for all platforms](https://github.com/cityjson/flatcitybuf/blob/main/src/cpp/INSTALL.md)
- [Comprehensive example](https://github.com/cityjson/flatcitybuf/blob/main/src/cpp/examples/comprehensive_example.cpp) — reading metadata, iterating features, spatial queries, writing, geometry & attribute access
