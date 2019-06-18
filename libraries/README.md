# New OSS C# Project Checklist

A list of tips for your C# libraries.

## The bare minimum

### Target netstandard

If possible, target `netstandard` rather than .NET Core or the full .NET Desktop Framework. This allows your package to be used by the widest number of consumers.

When targeting `netstandard`, target the lowest `netstandard` version you can. However, _be sure to follow the additional rules below_.

#### Netstandard < 2.0

If you target a `netstandard` version less than `2.0`, then multitarget that version along with `netstandard2.0`. This avoids the ["dozens of dependencies" problem](https://docs.microsoft.com/en-us/dotnet/standard/net-standard#which-net-standard-version-to-target).

#### Multitarget .NET 4.6.1

If you target `netstandard2.0`, also multitarget `net461`. This is because .NET 4.6.1 does not support NetStandard 2.0, even though it claims to. By mutlitargeting .NET 4.6.1, you [fix this problem for your consumers](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting).

### Provide XML documentation

XML docs give IntelliSense to your package consumers.

In your csproj:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

### Choose a license

Use [license expressions](https://docs.microsoft.com/en-us/nuget/reference/nuspec#license) if possible. This makes automated license auditing much easier.

E.g., for MIT, [in your csproj](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target):

```xml
<PropertyGroup>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
</PropertyGroup>
```

### Include a version

Use [semantic versioning](https://semver.org/). In your .csproj file, you can either keep the version and optional version suffix separate:

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

Used this way, version information is also applied to the dll. [More information about version properties](https://stackoverflow.com/a/42183301/263693).

### Metadata

You should have at least some [NuGet metadata](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#nuget-metadata-properties) defined in your .csproj, including `Authors`, `Description`, and `PackageProjectUrl`:

```xml
<PropertyGroup>
  <Authors>Me</Authors>
  <Description>My awesome library.</Description>
  <PackageProjectUrl>https://github.com/USER/PROJECT</PackageProjectUrl>
</PropertyGroup>
```

## Highly Recommended

### Icon

Projects with icons look more professional. [For now](https://github.com/NuGet/Home/issues/352), the best option is to upload your icon somewhere (e.g., in your GitHub repository) and include a link to it in your .csproj:

```xml
<PropertyGroup>
  <PackageIconUrl>https://raw.githubusercontent.com/USER/PROJECT/master/icon.png</PackageIconUrl>
</PropertyGroup>
```

### Enable Source Debugging

First, enable symbol packages. Use the [.snupkg format](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages-snupkg) rather than the [.symbols.nupkg format](https://docs.microsoft.com/en-us/nuget/create-packages/symbol-packages). This produces portable PDBs in a separate `.snupkg` file that is uploaded to NuGet.

In your csproj:

```xml
<PropertyGroup>
  <IncludeSymbols>true</IncludeSymbols>
  <SymbolPackageFormat>snupkg</SymbolPackageFormat>
</PropertyGroup>
```

Next, add [SourceLink debugging](https://github.com/dotnet/sourcelink). This will put information in your PDBs so that Visual Studio can step through the actual code of your project when your end-users are debugging.

For a project in a public GitHub repository, you should install the `Microsoft.SourceLink.GitHub` package and add the following to your csproj:

```xml
<PropertyGroup>
  <PublishRepositoryUrl>true</PublishRepositoryUrl>
</PropertyGroup>
```

## Nice to have

### Continuous integration

CI builds your project and runs your tests when you push to GitHub, and can be configured to publish NuGet packages as well.

Popular choices are [AppVeyor](https://www.appveyor.com/) and [Travis CI](https://travis-ci.com/), which are both free for open-source projects.

### Code coverage

First, you need a code coverage collector that detects code coverage when your unit tests are run. Popular choices are [coverlet](https://github.com/tonerdo/coverlet) and [OpenCover](https://github.com/OpenCover/opencover).

Once you have the collector going, you can upload the results to a website. Popular choices are [coveralls.io](https://coveralls.io/) and [codecov.io](https://codecov.io/), both free for open-source. Also see [ReportGenerator](https://github.com/danielpalme/ReportGenerator) for generating local code coverage reports.
