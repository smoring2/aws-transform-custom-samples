# Native Library Report

## Statically Bundled Libraries

| Library | Source JAR/Location | Architecture | Verdict | Resolution |
|---------|--------------------|--------------|---------|------------|
| — | — | — | — | No findings |

Scan performed per Phase 1.2.1. No JAR, WAR, fat/uber JAR or `.so` file exists anywhere in
the repository (`.git` excluded), so there was nothing to extract or inspect with `file`.

## Runtime-Extracted Libraries

| Dependency | Artifact | Extracts Native Code | ARM64 Support in Current Version | Verdict | Resolution |
|------------|----------|---------------------|----------------------------------|---------|------------|
| — | — | — | — | — | No findings |

Scan performed per Phase 1.2.2:

- `Native.load` / `Native.loadLibrary` / `System.loadLibrary` / `System.load(` in `*.java` — 0 matches (no Java sources).
- `jnr.` / `LibraryLoader` in `*.java` — 0 matches.
- `netty-transport-native`, `netty-tcnative`, `grpc-netty`, `conscrypt`, `jnr-ffi`, `jnr-posix`,
  `leveldbjni`, `rocksdbjni`, `sqlite-jdbc`, `lz4-java`, `zstd-jni`, `snappy-java` in
  `pom.xml` / `build.gradle` / `build.gradle.kts` — no such build files exist.

## Tiered Validation Policy Outcome

No library reached the FAIL, WARN or PASS determination because no native library candidates
were found. The tiered policy was not exercised.

## Resolution Details

No resolution was required. No `.so` recompilation, no ARM64 binary request to the user and
no native library loading logic change was necessary.

## Pure Java Fallbacks Documented

None. No native functionality exists that would require a fallback path.

## No Findings

No native libraries (bundled or runtime-extracted) detected. The repository contains no Java
artifacts of any kind — no `.java` sources, no `.jar` archives and no `.so` shared objects.

Note: `scaled-execution-containers/container/Dockerfile` compiles nothing native itself; it
installs `gcc`, `gcc-c++`, `make` and `*-devel` headers so that `pyenv` can build CPython from
source. Those toolchain packages are available on aarch64 in Amazon Linux 2023 and are
architecture-neutral in this Dockerfile.
