# migrator

One-shot job that applies the EF Core migrations to the shared PostgreSQL database, seeds data, then exits. The only component allowed to create or alter tables.

Part of Genkan. Architecture, endpoints and the full stack: see the **setup** repo.

## Layout

```
src/
  Genkan.Migrator/   (Migrations/, GenkanDbContext, Seeder)
Genkan.Migrator.slnx
```

## Build / run

`Genkan.Shared` comes from GitHub Packages, so restore needs a token with `read:packages`:

```bash
export GITHUB_PACKAGES_TOKEN=ghp_xxx
dotnet build Genkan.Migrator.slnx
docker build --secret id=github_token,env=GITHUB_PACKAGES_TOKEN -t migrator .
```

## Configuration (env)

`ConnectionStrings__Postgres`. Entities come from `Genkan.Shared`: an entity change = new Shared release **and** a new migration here.

```bash
dotnet tool install --global dotnet-ef
dotnet ef migrations add <Name> -p src/Genkan.Migrator -o Migrations
```

## CI / release

- push to `main` -> image `ghcr.io/genkan-software/migrator:main`, `:sha-xxxxxxx`, `:latest`
- tag `vX.Y.Z` -> also `:X.Y.Z` and `:X.Y`. Normally created for you: tagging **setup** releases every service with the same version.
- Pull requests only build.

Secrets: if `GITHUB_TOKEN` cannot read the `Genkan.Shared` package, either grant this repo access in the package settings
(Package settings > Manage Actions access) or add a `PACKAGES_TOKEN` secret (PAT, `read:packages`).
Dependabot (`.github/dependabot.yml`) opens a PR when a new `Genkan.Shared` is published; it needs `PACKAGES_TOKEN` as a *Dependabot* secret too.
