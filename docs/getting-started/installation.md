---
title: Installation
parent: Getting started
nav_order: 2
---

# Installation
{: .no_toc }

Where to actually get CrudApi running — this page is a pointer, not a repeat of the full guide.
{: .fs-6 .fw-300 }

---

Full, step-by-step deployment instructions live in their own section, since there's real detail to cover
for each path:

| You want to… | Go to |
|---|---|
| Run it in a container (the recommended path for most deployments) | [Deployment → Docker](../deployment/docker.html) |
| Know every environment variable / config key you can set | [Deployment → Environment variables](../deployment/environment-variables.html) |
| Understand the one-time first-run wizard that creates your first admin account and storage | [Deployment → Setup wizard](../deployment/setup-wizard.html) |
| Go to production — reverse proxy, HTTPS, backups, rollback | [Deployment → Production deployment](../deployment/production.html) |
| See how this repo builds and ships itself (for contributors, not required to self-host) | [Deployment → CI/CD](../deployment/azure-pipelines.html) |

The short version, for the impatient: pull the image, give it a volume for its data directory, start it,
and open it in a browser — the [setup wizard](../deployment/setup-wizard.html) takes it from there. See
[Deployment → Docker](../deployment/docker.html) for the exact `docker run`/compose shape, required volumes,
and port.

## Without Docker

Docker is the recommended path, but it isn't required. Every tagged release also publishes a plain
`.zip` of the published .NET binaries — see
[Releases](https://github.com/websvcin/crudapi/releases/latest):

```bash
wget https://github.com/websvcin/crudapi/releases/latest/download/crudapi-<version>.zip
unzip crudapi-<version>.zip -d crudapi
cd crudapi
dotnet CrudApi.dll
```

Requires the [.NET 8 runtime](https://dotnet.microsoft.com/download/dotnet/8.0) on the host. This is
the same path for Plesk or other shared hosting that doesn't offer Docker: upload the extracted
build and configure ASP.NET Core hosting per your provider's own instructions, setting the same
[environment variables](../deployment/environment-variables.html) Docker would otherwise set for you.

Once it's running, go to [Quickstart](quickstart.html) to create your first tenant and make your first API
call.
