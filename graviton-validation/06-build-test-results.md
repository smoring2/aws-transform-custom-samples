# Build & Test Validation Results

## Build Environment

- **Architecture:** `x86_64` (output of `uname -m`) — **not aarch64**
- **Java Runtime (default):** `openjdk version "25.0.3" 2026-04-21 LTS` / OpenJDK Runtime Environment Corretto-25.0.3.9.1
- **Java Runtime (Maven):** Corretto 17.0.19 at `/usr/lib/jvm/java-17-amazon-corretto.x86_64`
- **Build Tool:** Apache Maven 3.8.4 (Red Hat 3.8.4-3.amzn2023.0.5) — present on the host but
  with no project to build. No Gradle wrapper exists in this repository.
- **Docker:** not installed (`which docker` → not found)

### Build Environment Notes (Phase 3.0)

- **Java runtime alignment: not performed, and not needed.** The default runtime is Corretto 25
  while Maven already runs on Corretto 17. Alignment per Phase 3.0 is only required when a Java
  build is executed; there is no Java build here. No `JAVA_HOME` export was made, no subshell
  override was used, and no persistent environment change was made. The user's default
  `java -version` is unchanged (Corretto 25.0.3).
- **Annotation processors:** none. No `<annotationProcessorPaths>` and no `annotationProcessor`
  declaration exists, so the Lombok/MapStruct/Dagger Java-version sensitivities documented in
  Phase 3.0 do not apply.
- **Platform support:** this host is Linux, which is a fully supported platform for the
  transformation.

## Build Attempts

| # | Command | Result | Notes |
|---|---------|--------|-------|
| — | `mvn clean install` | ⬜ NOT RUN | No `pom.xml` anywhere in the repository |
| — | `./gradlew clean build` | ⬜ NOT RUN | No Gradle wrapper or build script |
| — | `docker buildx build --platform linux/arm64 -t app:arm64 .` | ⬜ NOT RUN | Docker not installed on this host; and the image is a tooling container, not an application artifact |

No build command was executed because there is no compilable Java unit. There is consequently
no "final build" in the conversation log for this repository, and no build score is meaningful.

## Test Failure Classification

| Test | Class | Failure Type | Root Cause | Blocking |
|------|-------|-------------|------------|----------|
| — | — | — | No Java tests exist | — |

Zero Java test sources (`*Test.java`, `@Test`) exist. The only automated test in the
repository is `scaled-execution-containers/cdk/test/cdk.test.ts`, a TypeScript Jest test for
the CDK stacks, which is not in scope for a Java ARM64 validation and was not run or modified.
No test file, test class or test method was removed or disabled by this transformation.

## Final Build (Scoring Basis)

- **Command:** none
- **Result:** ⬜ N/A
- **Rationale:** the repository contains no Java application, no Java build system and no Java
  tests. There is nothing to compile on ARM64, so neither a passing nor a failing build can be
  attributed to ARM64 compatibility.

## Container Validation (if applicable)

- **Build Command:** `docker buildx build --platform linux/arm64 -t atx-cli:arm64 scaled-execution-containers/container/`
- **Result:** ⬜ NOT VERIFIED — Docker is not installed on this validation host and the host is
  `x86_64`, so an aarch64 image build could not be attempted even under emulation.
- **Architecture Verified:** No. `os.arch = aarch64` was not confirmed empirically.
- **JDK Verified:** No. Amazon Corretto 17 remains the declared default in the Dockerfile and
  was not changed, but was not observed running on aarch64.

The Dockerfile changes described in `04-code-scan-findings.md` were reviewed statically:
`public.ecr.aws/amazonlinux/amazonlinux:2023` is a multi-arch manifest, AWS publishes
`awscli-exe-linux-aarch64.zip`, and Amazon Linux 2023 provides
`java-{8,11,17,21}-amazon-corretto-devel`, `maven`, `python3.11`–`python3.13` and the listed
`gcc`/`*-devel` packages for aarch64. Gradle (via SDKMAN), nvm/Node.js and pyenv all support
aarch64 Linux.

**Required user follow-up:** run the `docker buildx` command above on a Graviton instance, then:

```bash
docker run --rm --platform linux/arm64 --entrypoint java atx-cli:arm64 -version
# expect: OpenJDK Runtime Environment Corretto-17.x.x

docker run --rm --platform linux/arm64 --entrypoint java atx-cli:arm64 \
  -XshowSettings:properties -version 2>&1 | grep os.arch
# expect: os.arch = aarch64
```

Note the `--entrypoint java` override: this image declares
`ENTRYPOINT ["/app/entrypoint.sh"]`, so without the override the arguments would be appended
to that script instead of invoking `java`.

## Startup Validation

- **Application starts without errors:** ⬜ N/A — no Java application to start
- **JVM flags accepted:** ✅ Yes, for reference only. The JDK 17 Graviton flag set was accepted
  by Corretto 17 with exit code 0 — see `05-jvm-configuration.md`. No flags were applied to any
  workload, since no JVM workload exists.
- **Runtime crashes:** None observed; nothing was run.

## Performance Testing

Out of scope for this transformation and not applicable here. Should a Java workload be added
later, the user is responsible for benchmarking it on Graviton against their own baseline, at
production-like load, with Graviton driven to 70%+ CPU utilization for representative results.
