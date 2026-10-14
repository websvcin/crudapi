---
title: CI/CD
parent: Deployment
nav_order: 4
---

# CI/CD
{: .no_toc }

This repository's own build-and-release pipeline — relevant if you're self-hosting from source rather than
pulling the published image.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

CrudApi's own CI/CD runs on **Azure Pipelines** (`azure-pipelines.yml` in the repo root), triggered by a
push to `main` or a `v*.*.*` tag.

## What it does, in order

1. **Build & Publish .NET** — restores, builds in Release configuration, computes a version tag
   (`<majorMinor>.<buildId>` on a normal push, or the exact tag version on a `v*.*.*` push), publishes the
   app, and zips the output as a pipeline artifact.
2. **Docker — build & push to both registries** — builds the image once and pushes it to **both** GHCR
   and Docker Hub, tagged with the exact version, `latest`, and the current `<majorMinor>` series.
3. **GitHub Release** — downloads the build artifact and creates a GitHub Release with the versioned ZIP
   attached, a one-command `docker run` snippet in the release body, and links to both registries. Runs on
   **every** `main` push, not just tagged releases. A `422` (tag already exists — a rebuild) is treated as
   success rather than a failure.

## Required configuration (for a fork or your own instance of this pipeline)

| Needed | Where |
|---|---|
| `GITHUB_TOKEN` (PAT with `repo`, `write:packages`, `workflow`) and `GITHUB_USER` | Azure DevOps → Library → Variable group `github-secrets` |
| A Docker registry service connection to `ghcr.io` | Azure DevOps service connections, named `ghcr-connection` |
| A Docker registry service connection to Docker Hub | Azure DevOps service connections, named `dockerhub-connection` |

## Versioning

A normal push to `main` produces `<majorMinor>.<Build.BuildId>` (e.g. `1.0.47`) — the `majorMinor` value is
a variable at the top of the pipeline file, bumped manually to start a new minor series. Pushing a
`v*.*.*` tag instead uses that exact version.

## Tests

{: .known-issue }
> There is currently **no `dotnet test` step** in this pipeline, and no test project referenced by
> `Api.sln` for this repo's own local code (controllers, builders, bridge/extension classes). The private
> `WebSvc.*` packages this repo depends on each carry their own substantial xUnit test suites in their own
> repositories — the gap is specifically in this repo's own thin-host code, not the underlying platform
> logic. See [Reference → Known limitations](../reference/known-issues.html).

## If you're not using Azure Pipelines

None of this is required to self-host — pulling the published image (see [Docker](docker.html)) needs no
CI/CD of your own at all. This page documents how the published image you'd pull actually gets built and
released, for anyone contributing to or forking this repository specifically.
