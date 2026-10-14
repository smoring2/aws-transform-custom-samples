# Dependency ARM64 Compatibility Report

## Scope Note

This repository has no Java dependency manifest. `mvn dependency:tree` and
`./gradlew dependencies` could not be run because no `pom.xml`, `build.gradle`,
`build.gradle.kts` or `settings.gradle` exists. See `raw/dependency-tree-full.txt`
and `raw/dependency-tree-native.txt`.

Per `agent-scope-boundaries.md`, non-Java dependencies (npm, pip) are **out of scope** for
this transformation and were therefore not upgraded. They are listed below for information only.

## MUST UPGRADE (Blocking)

| Dependency | Current Version | Minimum ARM64 Version | Applied Version | ARM64 Issue | Evidence |
|------------|----------------|----------------------|-----------------|-------------|----------|
| — | — | — | — | — | No findings |

No blocking dependency upgrades were identified. No Java dependency exists to evaluate.

## RECOMMENDED UPGRADE (Non-Blocking)

| Dependency | Current Version | Recommended Version | Reason | Action Taken |
|------------|----------------|--------------------|---------|--------------|
| — | — | — | — | No findings |

## COMPATIBLE (No Action Needed)

| Dependency | Version | Basis |
|------------|---------|-------|
| — | — | No Java dependencies present |

## Transitive Dependency Resolutions

| Transitive Dependency | Pulled In By | Issue | Resolution Method |
|-----------------------|-------------|-------|-------------------|
| — | — | — | No findings |

No transitive native-code dependency was found, because there is no dependency tree to traverse.

## Non-Java Dependencies (Informational — Out of Scope)

These are recorded only so a reviewer can see that they were examined and intentionally
left unchanged.

| Manifest | Ecosystem | ARM64 Observation |
|---|---|---|
| `scaled-execution-containers/container/requirements.txt` | PyPI | Installed with `pip` inside the container; wheels or source builds resolve per architecture at build time. |
| `scaled-execution-containers/container/package.json` | npm | Dev tooling installed globally with `npm install -g`; architecture-neutral. |
| `scaled-execution-containers/api/lambda/requirements.txt` | PyPI | Lambda layer dependencies; unrelated to Graviton Java validation. |
| `scaled-execution-containers/cdk/package.json` | npm | AWS CDK TypeScript dependencies; architecture-neutral. |
| `agentic-atx-platform/sam/requirements-deploy.txt` | PyPI | Lambda deploy handler dependencies. |

No version in any of these manifests was modified.
