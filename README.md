# Jython (DataStax fork)

This is a DataStax fork of Jython, published to Artifactory as
`com.datastax.opscenter:jython-standalone` and consumed by OpsCenter (ripcord).

## Building and publishing

Jenkins job: https://datastax-team-opscenter-jenkins.swg-devops.com/job/jython-build-pipeline/

Trigger with parameter `buildbranch=<branch>`. The job builds
`jython-standalone-<version>.jar` with Ant and uploads it to:

```
https://repo.aws.dsinternal.org/artifactory/datastax-public-releases-local/com/datastax/opscenter/jython-standalone/<version>/
```

## Versioning

Versions are set in two files — both must be updated together before triggering a build:

- `build.xml` — `jython.version` and `jython.version.noplus` properties
- `maven/build.xml` — `project.version` property

### ⚠️ Use an alpha suffix when iterating

When developing or testing changes that are not yet ready to ship, use an alpha
suffix (e.g. `2.7.0.15a1`) rather than a plain version number. This prevents
Gradle from caching a broken artifact under the final version name. Promote to
the final version (e.g. `2.7.0.15`) only when the artifact is known good and
ready to be consumed by ripcord.

Background: Gradle caches dependency JARs by version. If a final version is
published, then republished with fixes under the same version number, CI nodes
that already downloaded the first artifact will use the stale cached copy and
fail — even though the correct artifact is in Artifactory. A version bump is
the only reliable way to force a cache invalidation. This bit us during
OPSC-17995, requiring multiple bumps (2.7.0.12 → 2.7.0.13 → 2.7.0.14 → 2.7.0.15).

## Known patches vs upstream

- **SNI support** (`f6ee77225`) — **reverted** in OPSC-17995. The SNI commit
  removed `NoVerifyX509TrustManager` from `_sslcerts.py`, breaking
  `opscenterd/src/opscenterd/SslUtils.py` in ripcord 6.8.x. Will be properly
  integrated as part of **OPSC-16690**.
- **commons-compress** — bumped from 1.10 → 1.27.1 in 2.7.0.13 to fix CVEs
  (OPSC-17995).

## Consuming in ripcord

Update `opscenterd/build.gradle` and `spock/build.gradle` in the ripcord repo:

```groovy
libs ("com.datastax.opscenter:jython-standalone:<version>")
```
