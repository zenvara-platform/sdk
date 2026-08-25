# Zenvara Connector Samples

Complete, buildable example connectors for [Zenvara](https://zenvara.dev) — the
AI-native automation platform that invokes YAML-defined flows as directed
acyclic graphs of actions.

A **connector** is a `.dll` the Zenvara runtime discovers through the
`[<ConnectorFactory>]` attribute and loads from the installed extension overlay.
Its schema — actions, input fields, entities, commands — is generated from a
`<name>.connector.yaml` manifest by the Schema Type Provider, so a typo in the
manifest is a compile error rather than a runtime surprise.

Copy either sample as the starting point for your own connector.

> This repository is vendored out of Zenvara's canonical source repository by a
> release job. Open issues here; pull requests against this mirror are
> overwritten on the next sync.

## The samples

| Sample | What it shows |
| --- | --- |
| `FSharp/` | An F# connector using the Schema Type Provider directly, plus its test project. The shortest path to a working connector. |
| `CSharp/` | A C# connector, the small F# bridge project that types its manifest (the Type Provider is F#-only), and its test project. |

Each builds, packs, and tests on its own.

## Building

The samples bind the SDK from nuget.org:

```bash
dotnet build FSharp/Zenvara.Connectors.FSharpSample/Zenvara.Connectors.FSharpSample.fsproj
dotnet test  FSharp/Zenvara.Connectors.FSharpSample.Tests/Zenvara.Connectors.FSharpSample.Tests.fsproj
```

## The SDK packages

| Package | What it is |
| --- | --- |
| `Zenvara.Sdk` | Meta-package. The single reference a connector adds. |
| `Zenvara.Sdk.Abstractions` | The frozen connector ABI — `Value`, `Payload`, `AppError`, `InvocationContext`, `ConnectorMetadata`. |
| `Zenvara.Sdk.Runtime` | Non-frozen runtime companion: helpers a connector uses but does not bind as contract. |
| `Zenvara.Sdk.Schema` | The Schema Type Provider. Types your `<name>.connector.yaml` manifest. |
| `Zenvara.Sdk.Services` | Host service seams. Not in the meta-package; reference it explicitly. |
| `Zenvara.Sdk.Testing` | Test helpers: a mock `InvocationContext`, a mock logger, a shared crash net. |

**F# connector** — one reference brings the frozen types and the Type Provider:

```xml
<PackageReference Include="Zenvara.Sdk" Version="3.0.0" />
```

**C# connector** — the Type Provider is F#-only, so a small F# bridge project
generates the typed schema and the C# project references the bridge:

```xml
<!-- MyConnector.Schema (F#) -->
<PackageReference Include="Zenvara.Sdk" Version="3.0.0" />

<!-- MyConnector (C#) -->
<ProjectReference Include="..\MyConnector.Schema\MyConnector.Schema.fsproj" />
```

`Zenvara.Sdk.Abstractions` is the **frozen** ABI: a connector compiled against it
stays loadable as the host moves forward. Its discriminated unions are
**additive** — new cases may appear, so match exhaustively and handle the unknown
case rather than assuming the set is closed. The other tiers carry no such
promise and may change between product versions.

## Installing a connector

Build your connector to a `.dll` and place it in the installed extension overlay
of your Zenvara installation (`Extensions:InstalledDirectory`). The runtime
discovers it on startup through the `[<ConnectorFactory>]` attribute — no
registration file, no restart hook to write.

## License

MIT. See [LICENSE.md](LICENSE.md).
