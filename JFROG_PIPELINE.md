# JFrog Integrated CI Pipeline

This repository includes a GitHub Actions workflow at `.github/workflows/jfrog-ci.yml` that satisfies the assignment requirements:

1. Checks out source code
2. Resolves build dependencies via JFrog Artifactory
3. Scans dependencies and artifact with JFrog Xray
4. Publishes the resulting artifact to JFrog Artifactory

## Trigger

- Manual run (`workflow_dispatch`) only

## JFrog Prerequisites

Create a free JFrog trial instance and configure these repositories:

- `maven-remote`: points to Maven Central
- `maven-local`: local Maven repository for published artifacts
- `maven-virtual`: virtual repository that includes `maven-remote` and `maven-local`

Recommended best-practice permissions:

- Use a dedicated technical user or access token for CI
- Grant read access to `maven-virtual`
- Grant deploy access to `maven-local`
- Keep least privilege and rotate tokens regularly

## GitHub Secrets

Add these repository secrets:

- `JF_URL`: your JFrog platform URL (example: `https://mycompany.jfrog.io`)
- `JF_ACCESS_TOKEN`: JFrog access token for CI

## How Dependency Resolution Is Forced Through JFrog

The workflow runs:

```bash
jf mvn-config \
  --repo-resolve-releases maven-virtual \
  --repo-resolve-snapshots maven-virtual \
  --repo-deploy-releases maven-local \
  --repo-deploy-snapshots maven-local
```

Then it builds with `jf mvn ...`, so Maven resolution and deployment are controlled by JFrog CLI configuration instead of public registries directly.

## Xray Scans

The pipeline performs:

- Dependency audit: `jf audit`
- Artifact scan: `jf scan target/*.jar`
- Optional build scan: `jf build-scan <build-name> <build-number>`

Scans are configured to keep the pipeline informative without blocking publication when known trial-tier limitations are detected (for example unavailable advanced scanners, limited build-scan permissions, or missing build indexing).

## Artifact Publication

The pipeline uploads built JAR files to a unique run-specific path:

`maven-local/org/springframework/samples/petclinic/<git-sha>/<run-id>-<run-attempt>/`

Build metadata is published with:

- `jf rt build-collect-env`
- `jf rt build-add-git`
- `jf rt build-publish`

This makes build and security context visible in the JFrog platform.

## Interview Demo Checklist

1. Show workflow run in GitHub Actions (`JFrog Integrated CI`)
2. Show dependency resolution through `maven-virtual`
3. Show Xray scan results in JFrog UI
4. Show uploaded JAR in `maven-local`
5. Show published build info in Artifactory/Xray
