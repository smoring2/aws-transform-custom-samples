# Graviton JVM Configuration

## Target Java Version

Not applicable as an application setting. There is no Java application and therefore no JVM
process launched by this repository — the container entrypoint is
`scaled-execution-containers/container/entrypoint.sh` (Bash), and the other two images run
Python. There is no `java -jar`, no `JAVA_OPTS`/`JAVA_TOOL_OPTIONS`, no systemd unit and no
`application.yml` in which JVM flags could be set.

The container's default tooling JDK is **Amazon Corretto 17** (`JAVA_HOME` in
`scaled-execution-containers/container/Dockerfile`), so the JDK 17 flag set from the
transformation definition is the one that would apply if a Java workload were ever added.

## Applied JVM Flags

| Flag | Applicable JDK Versions | Workload Type | Expected Impact |
|------|------------------------|---------------|-----------------|
| — | — | — | **None applied** |

No JVM flags were applied. Adding Graviton JVM tuning to a container whose workload is Bash,
Python and Node.js would have no effect and would be scope creep. Flags must be attached to a
real JVM workload to be meaningful.

## Flags NOT Applied (Version-Gated)

| Flag | Reason Not Applied |
|------|--------------------|
| `-XX:-TieredCompilation` | No JVM workload to tune |
| `-XX:ReservedCodeCacheSize=64M` | No JVM workload to tune |
| `-XX:InitialCodeCacheSize=64M` | No JVM workload to tune |
| `-XX:CICompilerCount=2` | No JVM workload to tune |
| `-XX:+UnlockDiagnosticVMOptions -XX:+UseAESCTRIntrinsics` | No JVM workload to tune; no cryptographic Java workload present |
| `-Xss1m` | No JVM workload to tune |
| `-XX:CompilationMode=high-only` | Withheld — removed in some JDK 21+ builds; the container also ships Corretto 21, so this flag is unsafe to standardize on here |

## Flag Validation

The JDK 17 Graviton flag set was nonetheless validated for acceptance against the Corretto 17
runtime available on this host, so that the recommendation below is known-good rather than
assumed:

```
$ /usr/lib/jvm/java-17-amazon-corretto.x86_64/bin/java \
    -XX:-TieredCompilation -XX:ReservedCodeCacheSize=64M -XX:InitialCodeCacheSize=64M \
    -XX:CICompilerCount=2 -XX:+UnlockDiagnosticVMOptions -XX:+UseAESCTRIntrinsics \
    -Xss1m -version

openjdk version "17.0.19" 2026-04-21 LTS
OpenJDK Runtime Environment Corretto-17.0.19.10.1 (build 17.0.19+10-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.19.10.1 (build 17.0.19+10-LTS, mixed mode, sharing)
exit=0
```

All flags were accepted with no warnings and no errors. Caveat: this host is `x86_64`, so this
confirms flag *recognition* by Corretto 17, not Graviton behaviour. Re-validate on an aarch64
Corretto 17 build before production use.

## Configuration Profiles

No profiles were created. If a Java workload is added to this repository later, apply the
JDK 17 profile above via `JAVA_TOOL_OPTIONS` or an explicit `java` command line, and re-run
`java <flags> -version` on the target aarch64 JDK first.
