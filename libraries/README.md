# New OSS C# Project Checklist

A list of tips for your C# libraries.

## The bare minimum

### Target netstandard

If possible, target `netstandard` rather than just .NET Core or the full .NET Desktop Framework. This allows your package to be used by the widest number of consumers.

When targeting `netstandard`, target the lowest `netstandard` version you can. However, _be sure to follow the additional rules below_.

#### Netstandard < 2.0

If you target a `netstandard` version less than `2.0`, then multitarget that version along with `netstandard2.0`. This avoids the ["dozens of dependencies" problem](https://docs.microsoft.com/en-us/dotnet/standard/net-standard#which-net-standard-version-to-target).

#### Multitarget .NET 4.6.1

If you target `netstandard2.0`, also multitarget `net461`. This is because .NET 4.6.1 does not support NetStandard 2.0, even though it claims to. By mutlitargeting .NET 4.6.1, you [fix this problem for your consumers](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting).

### Provide XML documentation

XML docs give IntelliSense to your package consumers.

In your project/props:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

### Choose a license

Use [license expressions](https://docs.microsoft.com/en-us/nuget/reference/nuspec#license) if possible. This makes automated license auditing much easier.

E.g., for MIT, [in your Directory.Build.props](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target):

```xml
<PropertyGroup>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
</PropertyGroup>
```

### Enable repeatable builds

When building in a CI environment, enable repeatable builds:

```xml
<PropertyGroup Condition="'$(CI)' == 'true'">
  <ContinuousIntegrationBuild>true</ContinuousIntegrationBuild>
</PropertyGroup>
```

### Include a version

Use [semantic versioning](https://semver.org/) and derive your version from tags pushed to the repository. If you are hosting on GitHub, you can set up your version to a specific value when building locally, and when GitHub Actions builds your project, pull your `Version` property from a tag starting with `v` as such:

```xml
<PropertyGroup Condition="'$(DevVersion)'==''">
  <DevVersion>1.0.0</DevVersion>
</PropertyGroup>
<PropertyGroup Condition="'$(GITHUB_REF)'=='' or !$(GITHUB_REF.StartsWith('refs/tags/v'))">
  <Version>$(DevVersion)-dev</Version>
</PropertyGroup>
<PropertyGroup Condition="'$(GITHUB_REF)'!='' and $(GITHUB_REF.StartsWith('refs/tags/v'))">
  <Version>$(GITHUB_REF.Replace('refs/tags/v', ''))</Version>
</PropertyGroup>
```

Used this way, version information is also applied to the dll. [More information about version properties](https://stackoverflow.com/a/42183301/263693).

#### Manual versions

Alternatively, you could maintain your version manually. In your project/props file, you can either keep the version and optional version suffix separate:

```xml
<PropertyGroup>
  <VersionPrefix>1.0.0</VersionPrefix>
  <VersionSuffix></VersionSuffix>
</PropertyGroup>
```

Or you can use a single combined property:

```xml
<PropertyGroup>
  <Version>1.0.0</Version>
</PropertyGroup>
```

Whichever is easier for you and/or your release scripts to change.

### Enable code quality checks

All builds should have warnings-as-errors:

```xml
<PropertyGroup>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
</PropertyGroup>
```

Modern libraries should adopt nullable types:

```xml
<PropertyGroup>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

And FxCop is still alive and relevant:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.CodeAnalysis.FxCopAnalyzers" Version="3.0.0">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
</ItemGroup>
```

FxCop can be a bit too pedantic. Control FxCop settings with an `.editorconfig` file:

```ini
[*.cs]

# CA1063: Implement IDisposable correctly
dotnet_diagnostic.CA1063.severity = silent
```

### Metadata

You should have at least some [NuGet metadata](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#nuget-metadata-properties) defined in your [csproj/props](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target), including `Authors`, `Description`, `PackageTags`, and `PackageProjectUrl`:

```xml
<PropertyGroup>
  <Authors>Me</Authors>
  <Description>My awesome library.</Description>
  <PackageTags>tag1;tag2</PackageTags>
  <PackageProjectUrl>https://github.com/$(GITHUB_REPOSITORY)</PackageProjectUrl>
</PropertyGroup>
```

## Highly Recommended

### Icon

Projects with icons look more professional. Add an icon to your repository (the example below assumes it is at `./src/icon.png`) and include this in your props:

```xml
<PropertyGroup>
  <PackageIcon>icon.png</PackageIcon>
</PropertyGroup>
<ItemGroup>
  <None Include="..\icon.png" Pack="true" PackagePath="\"/>
</ItemGroup>
```

### Enable Source Debugging

First, enable symbol packages. Use the [.snupkg format](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg) rather than the [.symbols.nupkg format](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages). This produces portable PDBs in a separate `.snupkg` file that is uploaded to NuGet.

In your csproj/props:

```xml
<PropertyGroup>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
  <EmbedUntrackedSources>true</EmbedUntrackedSources>
  <PublishRepositoryUrl>true</PublishRepositoryUrl>
</PropertyGroup>
```

Next, add [SourceLink debugging](https://github.com/dotnet/sourcelink). This will put information in your PDBs so that Visual Studio can step through the actual code of your project when your end-users are debugging. For a project in a public GitHub repository, you should install the `Microsoft.SourceLink.GitHub` package in your csproj/props:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.SourceLink.GitHub" Version="1.0.0" PrivateAssets="All"/>
</ItemGroup>
```

## Nice to have

### Code coverage

First, you need a code coverage collector that detects code coverage when your unit tests are run. Popular choices are [coverlet](https://github.com/tonerdo/coverlet) and [OpenCover](https://github.com/OpenCover/opencover).

Once you have the collector going, you can upload the results to a website. Popular choices are [coveralls.io](https://coveralls.io/) and [codecov.io](https://codecov.io/), both free for open-source. Also see [ReportGenerator](https://github.com/danielpalme/ReportGenerator) for generating local code coverage reports.
