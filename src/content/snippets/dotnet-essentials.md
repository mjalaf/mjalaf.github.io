---
title: ".NET Core / .NET CLI Essentials"
description: "Cross-version cheatsheet for .NET CLI — projects, packages, EF Core, testing, publishing, and version differences from .NET Core 3.x through .NET 10."
category: ".NET"
---

## SDK & runtime management

```bash
# What's installed?
dotnet --info
dotnet --list-sdks
dotnet --list-runtimes

# Pin an SDK for a repo (creates ./global.json)
dotnet new globaljson --sdk-version 8.0.100 --roll-forward latestFeature

# Install/uninstall on Linux/macOS via the official install script
curl -sSL https://dot.net/v1/dotnet-install.sh | bash -s -- --channel 9.0
```

## Project & solution scaffolding

```bash
# List available templates
dotnet new list

# Common templates
dotnet new console        -o MyApp
dotnet new webapi         -o MyApi --use-controllers   # classic controllers
dotnet new webapi         -o MyApi                     # minimal APIs (.NET 6+)
dotnet new mvc            -o MyMvc
dotnet new blazor         -o MyBlazor                  # unified template (.NET 8+)
dotnet new worker         -o MyWorker
dotnet new classlib       -o MyLib
dotnet new xunit          -o MyLib.Tests

# Solution wiring
dotnet new sln -n MySolution
dotnet sln add MyApi/MyApi.csproj MyLib/MyLib.csproj
dotnet add MyApi reference ../MyLib/MyLib.csproj
```

## Build, run, watch, test

```bash
dotnet restore
dotnet build -c Release
dotnet run --project MyApi
dotnet watch --project MyApi run        # hot reload (.NET 6+ default behavior in 'watch run')

dotnet test
dotnet test --collect:"XPlat Code Coverage" --logger "trx;LogFileName=results.trx"
dotnet test --filter "Category=Integration"
```

## NuGet packages

```bash
# Add / remove / list / update
dotnet add package Serilog.AspNetCore --version 8.0.3
dotnet remove package Serilog.AspNetCore
dotnet list package
dotnet list package --outdated
dotnet list package --vulnerable --include-transitive

# Tool packages
dotnet tool install -g dotnet-ef
dotnet tool update  -g dotnet-ef
dotnet tool restore               # from .config/dotnet-tools.json (local tools)
```

## Entity Framework Core

```bash
# Migrations
dotnet ef migrations add InitialCreate --project MyApi.Data --startup-project MyApi
dotnet ef database update
dotnet ef migrations remove
dotnet ef migrations script -i -o ./migrate.sql

# Scaffold a model from an existing DB
dotnet ef dbcontext scaffold "Server=...;Database=...;User=...;Password=...;Encrypt=true" \
  Microsoft.EntityFrameworkCore.SqlServer -o Models -c AppDbContext
```

## Publishing

```bash
# Framework-dependent (smaller, needs runtime installed)
dotnet publish -c Release -o ./publish

# Self-contained, single-file, trimmed (great for containers/CLIs)
dotnet publish -c Release \
  -r linux-x64 \
  --self-contained true \
  -p:PublishSingleFile=true \
  -p:PublishTrimmed=true \
  -p:InvariantGlobalization=true

# Native AOT (.NET 7+, recommended .NET 8+/9+ for APIs)
dotnet publish -c Release -r linux-x64 -p:PublishAot=true

# Container image (.NET 7+ via Microsoft.NET.Build.Containers; built-in in .NET 8+)
dotnet publish --os linux --arch x64 /t:PublishContainer -c Release
```

## User secrets & configuration

```bash
# Initialize secrets storage for a project
dotnet user-secrets init --project MyApi
dotnet user-secrets set "ConnectionStrings:Default" "Server=...;Database=..."
dotnet user-secrets list
dotnet user-secrets remove "ConnectionStrings:Default"

# Environment for dotnet run
ASPNETCORE_ENVIRONMENT=Development dotnet run
```

## Minimal API (one-file service, .NET 6+)

```csharp
// Program.cs — .NET 6 introduced minimal hosting; refined through .NET 8/9.
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddOpenApi();              // .NET 9+: built-in OpenAPI (replaces Swashbuckle)

var app = builder.Build();
app.MapOpenApi();
app.MapGet("/health", () => Results.Ok(new { status = "ok" }));
app.MapGet("/users/{id:int}", (int id) => new { id, name = "Martin" });
app.Run();
```

## Version differences at a glance

| Version | Released | LTS? | Notable |
| --- | --- | --- | --- |
| .NET Core 3.1 | Dec 2019 | (EOL Dec 2022) | gRPC, Worker Services, last "Core" branding |
| .NET 5     | Nov 2020 | No  | Unified `.NET` branding, no more "Core" |
| .NET 6     | Nov 2021 | **Yes** (EOL Nov 2024) | Minimal APIs, hot reload, `record struct`, file-scoped namespaces |
| .NET 7     | Nov 2022 | No  | Native AOT preview, rate limiting, generic math |
| .NET 8     | Nov 2023 | **Yes** | Native AOT GA, container publish built-in, primary constructors, Blazor United |
| .NET 9     | Nov 2024 | No  | Built-in OpenAPI, hybrid cache, faster GC, `params` collections |
| .NET 10    | Nov 2025 | **Yes** | Latest LTS; refined AOT, performance, language features (C# 14) |

> Always target an LTS version (6/8/10) for production unless you specifically need a current release. Use `global.json` to pin per-repo.

## Migration tips (Core 3.x → modern .NET)

- Replace `Startup.cs` with the minimal hosting model in `Program.cs` (or keep `Startup` — both still work).
- `Newtonsoft.Json` → `System.Text.Json` (faster, source-generator friendly). Add `Microsoft.AspNetCore.Mvc.NewtonsoftJson` only if you need feature parity.
- `IHostingEnvironment` → `IWebHostEnvironment` / `IHostEnvironment`.
- `IHttpClientFactory` is the right way to consume HTTP — keep it.
- For containers: switch to `dotnet publish /t:PublishContainer` (no Dockerfile needed for typical APIs).
- Run `dotnet format` and enable nullable reference types (`<Nullable>enable</Nullable>`) project by project.

## Diagnostics

```bash
# CLI diagnostics tools (install once)
dotnet tool install -g dotnet-counters
dotnet tool install -g dotnet-trace
dotnet tool install -g dotnet-dump
dotnet tool install -g dotnet-gcdump

# Live counters for a running process
dotnet-counters monitor --process-id <PID> \
  System.Runtime Microsoft.AspNetCore.Hosting

# Collect a trace
dotnet-trace collect --process-id <PID> --providers Microsoft-DotNETCore-SampleProfiler

# Capture a memory dump
dotnet-dump collect --process-id <PID>
```
