---
layout: default
title: C++
parent: FlatCityBuf
nav_order: 5
has_children: false
permalink: /flatcitybuf/cpp/
---

# Using FlatCityBuf with C++

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The C++ library is a from-scratch **native C++17 implementation** that reads *and* writes FlatCityBuf. It replaces the earlier CXX-bridge bindings over the Rust core: there is no Rust toolchain to install, no generated bridge source to compile, no async runtime, and — unless you ask for the HTTP adapter — no TLS dependency.

Source: [`src/cpp`](https://github.com/cityjson/flatcitybuf/tree/main/src/cpp). The rendered API reference is at [cityjson.github.io/flatcitybuf/cpp](https://cityjson.github.io/flatcitybuf/cpp/); the canonical guide — build options, the `RangeReader` contract — is [docs/cpp.md](https://github.com/cityjson/flatcitybuf/blob/main/docs/cpp.md).

## Dependencies

| Dependency | Required? | Why |
| --- | --- | --- |
| `flatbuffers` | yes | the on-disk format |
| `nlohmann-json` | with `FCB_WITH_JSON=ON` (the default) | CityJSON emission |
| `libcurl` | with `FCB_WITH_CURL=ON` (default **OFF**) | HTTP range requests |
| `doctest` | with `FCB_BUILD_TESTS=ON` (the default) | tests only, never installed |

{% raw %}

```bash
# macOS
brew install flatbuffers nlohmann-json doctest

# Debian / Ubuntu
sudo apt-get install libflatbuffers-dev nlohmann-json3-dev doctest-dev
```

{% endraw %}

## Installing with vcpkg

The `flatcitybuf` port lives in a [custom vcpkg registry](https://github.com/HideBa/vcpkg), not the built-in microsoft/vcpkg one. Next to your project's `vcpkg.json`, add a `vcpkg-configuration.json`:

{% raw %}

```json
{
  "default-registry": {
    "kind": "git",
    "repository": "https://github.com/microsoft/vcpkg",
    "baseline": "2f1d605400c8727cc00c15797aba796c88ccd523"
  },
  "registries": [
    {
      "kind": "git",
      "repository": "https://github.com/HideBa/vcpkg",
      "baseline": "9171cc35ccca68dc481c7a3718d7785a2fb5c20e",
      "packages": ["flatcitybuf"]
    }
  ]
}
```

{% endraw %}

and declare the dependency in `vcpkg.json`:

{% raw %}

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "dependencies": ["flatcitybuf"]
}
```

{% endraw %}

Use `{ "name": "flatcitybuf", "features": ["curl"] }` instead to get the HTTP range-request reader. Configure with vcpkg's toolchain file (`-DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake`) and integrate exactly as in [the CMake snippet below](#building-and-installing) — the port installs the same `flatcitybuf::flatcitybuf` target the manual build does.

The baselines above pin what you get (currently port 0.9.0, built from the `cpp-v0.9.0` tag); they move with each release, and [`src/cpp/INSTALL.md`](https://github.com/cityjson/flatcitybuf/blob/main/src/cpp/INSTALL.md#install-via-vcpkg) always records the current ones. To pick up a newer release later, bump the registry baseline to a newer commit of the registry repository.

## Building and installing

{% raw %}

```bash
git clone https://github.com/cityjson/flatcitybuf.git
cd flatcitybuf/src/cpp

cmake -B build -S .
cmake --build build
cmake --install build --prefix /your/prefix
```

{% endraw %}

Useful options: `-DFCB_WITH_CURL=ON` (HTTP support), `-DFCB_WITH_JSON=OFF` (drop CityJSON emission and the nlohmann dependency), `-DFCB_BUILD_TESTS=OFF`, `-DFCB_BUILD_EXAMPLES=OFF`.

Then, from your own CMake project:

{% raw %}

```cmake
find_package(flatcitybuf CONFIG REQUIRED)
target_link_libraries(my_app PRIVATE flatcitybuf::flatcitybuf)
```

{% endraw %}

That is the whole integration — the FlatBuffers headers are generated and committed, so consumers never need `flatc`.

## Reading a local file

{% raw %}

```cpp
#include <fcb/cityjson.hpp>
#include <fcb/reader.hpp>

#include <iostream>

int main() {
    fcb::FcbReader reader = fcb::FcbReader::open_file("delft.fcb");

    const auto& info = reader.header().info();
    std::cout << info.features_count << " features, CityJSON "
              << info.cityjson_version << ", " << info.crs << "\n";

    // One CityJSONSeq metadata line, then one CityJSONFeature per line.
    std::cout << fcb::to_cityjson_metadata(reader.header()).dump() << "\n";

    auto it = reader.select_all();
    while (it.next()) {
        std::cout << fcb::to_cityjson_feature(it.current(), reader.header()).dump() << "\n";
    }
}
```

{% endraw %}

`to_cityjson_metadata` and `to_cityjson_feature` return `nlohmann::json`, so field access is the ordinary nlohmann API: `.at("k")`, `.value("k", default)`, `.contains("k")`, `.get<double>()`.

## Spatial queries

{% raw %}

```cpp
auto it = reader.select_bbox({84227.77, 445377.33, 85323.23, 446334.69});
while (it.next()) {
    std::cout << fcb::to_cityjson_feature(it.current(), reader.header())["id"] << "\n";
}
```

{% endraw %}

## Attribute queries

`select_attr` uses the static B+trees, so the column must have been indexed at write time. The comparison value is a typed `KeyValue` and **its type must match the column's type on disk** — a mismatch does not throw, it reinterprets bytes.

{% raw %}

```cpp
#include <fcb/stree.hpp>

fcb::AttrQuery query = {
    {"b3_h_dak_50p", fcb::Operator::Gt, fcb::KeyValue::from_f64(20.0)},
};

auto it = reader.select_attr(query);
while (it.next()) {
    // 4 of 1115 features match in the Delft example file
}
```

{% endraw %}

String columns are indexed on keys truncated to 50 bytes (100 for JSON/binary columns), so the index returns candidates; the default `AttrQueryOptions` verify each one against the fully decoded attribute. Pass `{true}` to skip verification — faster, and wrong for long strings.

## Reading over HTTP

Build with `-DFCB_WITH_CURL=ON`:

{% raw %}

```cpp
#include <fcb/http/curl_range_reader.hpp>

auto transport = std::make_shared<fcb::CurlRangeReader>(
    "https://flatcitybuf.open3d.city/data/3dbag_all_index.fcb");
fcb::FcbReader reader = fcb::FcbReader::open(transport);

auto it = reader.select_bbox({120000, 486000, 121000, 487000});
```

{% endraw %}

Only the intersecting features are fetched. On the published 3DBAG file (~68GB, 10.7M features) the example program opens the file in **2 HTTP requests** and answers a 1km bounding box in **37**.

## Bringing your own transport

`fcb::RangeReader` is the library's only IO seam — implement it to read from an object store, a game-engine VFS, an mmap, memory, or a decrypting layer:

{% raw %}

```cpp
class MyReader : public fcb::RangeReader {
    std::uint64_t total_size() override { /* ... */ }
    std::vector<std::uint8_t> read(std::uint64_t offset, std::uint64_t length) override { /* ... */ }
    // Optionally override read_batch() to pipeline or multiplex.
};
```

{% endraw %}

The interface is deliberately **synchronous**: batching, not asynchrony, is the concurrency primitive, and a blocking interface is trivially wrapped by whatever threading model your application already has. Read the contract comment in `include/fcb/range_reader.hpp` before implementing one.

## Writing files

`fcb::FcbWriter` writes `.fcb` from CityJSON-shaped JSON, with no Rust toolchain involved. Its output is validated byte-for-byte against files written by the Rust writer.

{% raw %}

```cpp
#include <fcb/writer/attribute.hpp>
#include <fcb/writer/fcb_writer.hpp>

// `cj` is the CityJSONSeq metadata line; the schemas must already describe
// every feature that will be added, so scan all features first (column
// numbering is insertion order, exactly as the Rust CLI does it).
fcb::FcbWriter writer(cj, options, attr_schema, semantic_attr_schema);

for (const auto& feature : features) {
    writer.add_feature(feature); // spooled to a temp file, not kept in memory
}

std::ofstream out("city.fcb", std::ios::binary);
writer.write(out); // streams header, indices and features straight to `out`
```

{% endraw %}

`add_feature` spools each encoded feature to a private temporary file and `write(std::ostream&)` streams the finished file out in fixed-size chunks, so memory stays bounded regardless of dataset size. (There is also a `write()` overload returning a `std::vector<std::uint8_t>`; it is a convenience for small files and does *not* have that property.)

## Examples

The repository ships nine self-contained example programs, one per capability — see [`src/cpp/examples`](https://github.com/cityjson/flatcitybuf/tree/main/src/cpp/examples), which documents the exact output of each:

| Program | Shows |
| --- | --- |
| `fcb_inspect_header` | header only: extent, CRS, transform, and which columns are queryable |
| `fcb_read_local` | the whole file (or a bbox) as CityJSONSeq |
| `fcb_to_cityjson` | the CityJSON representation, and how to reach into its fields |
| `fcb_query_attributes` | attribute queries through the B+tree |
| `fcb_read_features` | raw feature access, without CityJSON conversion |
| `fcb_custom_reader` | implementing `fcb::RangeReader` yourself |
| `fcb_read_http` | remote reads over HTTP range requests |
| `fcb_geometry_analysis` | walking the encoded geometry directly, for analysis |
| `fcb_write_cityjson` | writing a CityJSONSeq out as `.fcb` |

`fcb_custom_reader` is the one that makes the format's argument concrete: on the Delft file, reading everything costs 7 reads and 90.7% of the bytes, while a bounding-box query costs 4 reads and 31.7% of the bytes for 170 of 1115 features.

`fcb_geometry_analysis` is the one that skips CityJSON entirely: `fcb::Feature::raw()` is public, and returns the generated `CityFeature` table holding the **encoded** geometry — the format's own flat count arrays (`solids`/`shells`/`surfaces`/`strings`, plus the flat `boundaries` index list) and the quantised vertices they index into. That is the cheapest representation to compute over, since nothing has to be nested, allocated or turned into JSON; the example sums surface area per semantic surface type straight from it. Note that nesting depth comes from `Geometry::type()`, never from which array is populated — a `Solid` with one shell and a `MultiSolid` with one solid flatten to byte-identical arrays.

## Verification

The reader's output is compared against the Rust reader on the full Delft fixture (all 1115 features, compared as parsed JSON trees), plus the shared [conformance corpus](https://github.com/cityjson/flatcitybuf/tree/main/conformance) — single-feature files, prefix-colliding strings, duplicate keys, zero-area extents, geometry templates, appearance at every nesting depth. The test suite runs clean under ASan and UBSan.
