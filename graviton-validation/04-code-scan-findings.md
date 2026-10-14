# Architecture-Specific Code Scan

## Architecture Detection Patterns Found

### Java sources

| File | Line(s) | Pattern | ARM64 Handling Present | Action Required |
|------|---------|---------|----------------------|-----------------|
| — | — | — | — | No findings |

Scanned for `System.getProperty("os.arch")`, `System.getProperty("os.name")`,
`Native.load()` and `System.loadLibrary()` across `*.java`. Zero matches — the repository
contains no Java source files.

### Build and container configuration

The repository-wide scan for `x86_64`, `amd64`, `aarch64`, `arm64` and `--platform` did
surface hardcoded x86 assumptions in container and deployment configuration:

| File | Line(s) | Pattern | ARM64 Handling Present | Action Taken |
|------|---------|---------|----------------------|--------------|
| `scaled-execution-containers/container/Dockerfile` | `FROM public.ecr.aws/amazonlinux/amazonlinux:2023` | Base image with no `--platform` annotation | No | **Fixed** — added `--platform=$BUILDPLATFORM` |
| `scaled-execution-containers/container/Dockerfile` | AWS CLI install | `awscli-exe-linux-x86_64.zip` hardcoded | No | **Fixed** — now `awscli-exe-linux-$(uname -m).zip` |
| `scaled-execution-containers/container/Dockerfile` | `ENV JAVA_HOME` | `/usr/lib/jvm/java-17-amazon-corretto.x86_64` hardcoded | No | **Fixed** — arch-independent symlink built from `$(uname -m)` |
| `scaled-execution-containers/deployment/1-build-and-push.sh` | 123, 125 | `docker build --platform linux/amd64` | No | **Not changed** — deployment automation is out of scope; reported below |
| `scaled-execution-containers/cdk/lib/container-stack.ts` | 34 | `platform: ecrAssets.Platform.LINUX_AMD64` | No | **Not changed** — infrastructure is out of scope; reported below |
| `agentic-atx-platform/cdk/lib/container-stack.ts` | 36 | `platform: ecrAssets.Platform.LINUX_AMD64` | No | **Not changed** — infrastructure is out of scope; reported below |

`agentic-atx-platform/sam/deploy.sh:60` already builds the orchestrator image with
`--platform linux/arm64`, so that path is Graviton-ready and needed no change.

## JNI/JNA Usage

| File | Line(s) | Type | Library | ARM64 Validated |
|------|---------|------|---------|-----------------|
| — | — | — | — | No findings |

No JNI or JNA usage exists. Searches for `jnr.`, `LibraryLoader`, `Native.load` and
`System.load(` returned zero matches.

## Changes Applied

Phase 2.3 made no Java code changes, because there is no Java code. One file was modified,
limited strictly to removing x86-only assumptions so that the Java-bearing tooling container
can be built on aarch64:

`scaled-execution-containers/container/Dockerfile`

1. **Base image platform annotation.** Single-stage Dockerfile, so `--platform=$BUILDPLATFORM`
   was added per Phase 2.4. `public.ecr.aws/amazonlinux/amazonlinux:2023` is a multi-arch
   manifest and resolves to the aarch64 variant automatically on a Graviton builder.
2. **AWS CLI v2 download URL.** Changed to `awscli-exe-linux-$(uname -m).zip`. `uname -m`
   yields `x86_64` or `aarch64`, and AWS publishes the installer under both names, so the same
   line works on either architecture.
3. **`JAVA_HOME`.** Amazon Corretto RPMs install into an architecture-suffixed directory
   (`java-17-amazon-corretto.x86_64` vs `java-17-amazon-corretto.aarch64`). A stable symlink
   `/usr/lib/jvm/java-17-amazon-corretto-current` is created from `$(uname -m)` and `JAVA_HOME`
   now points at it. Without this, `JAVA_HOME` would dangle on aarch64 and every Maven and
   Gradle invocation inside the container would fail.

Explicitly **not** changed, to stay inside the transformation scope:

- The JDK distribution remains Amazon Corretto and the default version remains 17.
- Corretto 8/11/17/21, Maven, Gradle 8.5, Python and Node.js version selections are untouched.
- `.gitignore` and `.dockerignore` were not created or modified.
- No deployment script or CDK stack was modified (infrastructure migration and deployment
  automation are out of scope). These remain pinned to `linux/amd64` and are the user's
  responsibility to switch when they choose to deploy on Graviton — see `00-summary.md`.
