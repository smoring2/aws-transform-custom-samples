# Project Assessment

## Deployment Type

**Containerized** (tooling/automation containers only — no Java application artifact).

Evidence:
- `scaled-execution-containers/container/Dockerfile` — Amazon Linux 2023 multi-language
  AWS Transform CLI runner. Installs Amazon Corretto 8/11/17/21, Maven, Gradle 8.5,
  Python 3.8–3.13 and Node.js 16–24. Entrypoint is `/app/entrypoint.sh` (Bash), not a JVM process.
- `agentic-atx-platform/orchestrator/Dockerfile` — Python 3.11 agent (`python agent.py`).
- `agentic-atx-platform/sam/Dockerfile.deploy` — Python 3.11 Lambda handler.

No systemd/init units, no JAR/WAR deployment, no Java service definition exists.

## Project Structure

- **Build Tool:** None for Java. No Maven and no Gradle project is present.
- **Multi-Module:** No.
- **Submodules:** Not applicable.

File inventory (repository root, `.git` excluded, 158 files total):

| Artifact searched | Matches |
|---|---|
| `pom.xml` | 0 |
| `build.gradle` / `build.gradle.kts` | 0 |
| `settings.gradle` / `settings.gradle.kts` | 0 |
| `*.java` | 0 |
| `*.jar` | 0 |
| `*.so` | 0 |
| `Dockerfile*` | 3 (see above) |

Actual content by extension: 51 Markdown, 23 Bash, 21 Python, 14 JSON, 13 TypeScript,
8 JSX, 2 JavaScript, plus assets. The repository is a collection of AWS Transform
transformation definitions and the automation/infrastructure used to execute them at scale.

## Java Environment

No Java application exists in this repository, so there is no application Java version
to record. The Java runtimes that appear are tooling installs inside the execution container:

- **Java Versions (container tooling):** Amazon Corretto 8, 11, 17, 21; default Corretto 17.
- **JDK Distribution:** Amazon Corretto (unchanged — not modified by this transformation).
- **ARM64 Support Status:** Supported. Amazon Corretto publishes aarch64 builds for 8, 11, 17 and 21,
  and all four are available from the Amazon Linux 2023 `dnf` repositories on aarch64.
- **Known Graviton Issues:** None for Corretto 8/11/17/21 on aarch64.

Validation-host environment (for the record — this host is **not** Graviton):

- `uname -m` → `x86_64`
- `java -version` → OpenJDK Corretto 25.0.3.9.1
- `mvn -version` → Apache Maven 3.8.4, running on Corretto 17.0.19
- `docker` → not installed on this host

## Component Risk Categorization

| Component | Risk Level | Rationale |
|-----------|-----------|-----------|
| Java application source | N/A | No Java source files exist in the repository. |
| Native libraries / JNI / JNA | N/A | No `.so` files and no JAR files exist. |
| Java dependency manifests | N/A | No `pom.xml` / `build.gradle` exists. |
| `scaled-execution-containers/container/Dockerfile` | MEDIUM | Hardcoded `x86_64` in the AWS CLI download URL and in `JAVA_HOME`; would fail an aarch64 build. Resolved — see `04-code-scan-findings.md`. |
| `scaled-execution-containers/deployment/1-build-and-push.sh` | MEDIUM | Pins `docker build --platform linux/amd64`. Deployment automation — reported, not changed (out of scope). |
| `scaled-execution-containers/cdk/lib/container-stack.ts`, `agentic-atx-platform/cdk/lib/container-stack.ts` | MEDIUM | `platform: ecrAssets.Platform.LINUX_AMD64` and Fargate `X86_64` runtime. Infrastructure — reported, not changed (out of scope). |
| Markdown transformation definitions | LOW | Documentation; architecture-independent. |
| Python / TypeScript / Bash sources | LOW | Not Java; outside the scope of this transformation. |

## Entry Criteria Assessment

| # | Entry Criterion | Met | Note |
|---|---|---|---|
| 1 | Java application running on x86 | ❌ No | No Java application in repository |
| 2 | Source code and build scripts available | ⚠️ Partial | Build scripts exist, but no Java build |
| 3 | Current Java version documented | ⚠️ N/A | Container tooling only (Corretto 8/11/17/21) |
| 4 | Build/deployment configuration available | ✅ Yes | Dockerfiles, CDK, deployment scripts |
| 5 | ARM64 build/test environment | ❌ No | Host is `x86_64`; Docker not installed |
| 6 | Complete dependency list | ❌ No | No Java dependency manifest to analyze |
| 7 | Existing test suite runnable on ARM64 | ❌ No | No Java test suite (`cdk.test.ts` is TypeScript) |

**Conclusion:** the entry criteria for a Java x86-to-Graviton migration are not satisfied.
Phases 1.2–1.5, 2.1–2.3, 2.5 and 3.1–3.3 have no applicable Java subject matter.
