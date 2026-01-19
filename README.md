# Build Visual Studio .NET Project NuGet Release Action

This GitHub Action builds a Visual Studio .NET project, packages the build artifacts into a NuGet package (`.nupkg`), creates a semantic version tag, and publishes a GitHub Release (with optional pre-release support). It is designed to automate the process of building, packaging, and releasing .NET projects as distributable NuGet packages.

---

## Features

- Builds a Visual Studio .NET project (e.g., .csproj, .vbproj) using `dotnet pack` and generates a versioned NuGet package.
- Supports both standard and pre-release NuGet versioning with a custom suffix.
- Creates an annotated git tag for new versions.
- Publishes a GitHub Release and attaches the `.nupkg` file (if not a pre-release).
- Adds a job summary with release details.
- Comments the release status and tag on the associated pull request.
- Includes the release date and time (Central Daylight Time, CDT) in the release notes.

---

## Inputs

| Name                | Description                                                                 | Required |
|---------------------|-----------------------------------------------------------------------------|----------|
| `project-file-path` | The path to the Visual Studio .NET project file (e.g., .csproj, .vbproj) to be built. | Yes      |
| `prerelease`        | Mark the release as a pre-release (`true`/`false`).                          | Yes      |
| `prerelease-suffix` | The suffix to append to the version number for pre-releases. For non-pre-releases, the version is based on the latest pre-release with this suffix. | Yes      |
| `package-name`      | The base name for the NuGet package file containing the build artifacts.     | Yes      |
| `configuration`     | The build configuration to use for the .NET project (e.g., Debug, Release).  | Yes      |
| `token`             | The GitHub token used for authentication to create the release.              | Yes      |

---

## Outputs

| Name              | Description                                                          |
|-------------------|----------------------------------------------------------------------|
| `version-tag`     | The new tag created for the release (e.g., `v1.0.0`).                |
| `version-number`  | The new tag with the "v" prefix removed (e.g., `1.0.0`).             |
| `nuget-file-name` | The name of the NuGet file containing the build artifacts, including the `.nupkg` extension (e.g., `my-package.1.0.0.nupkg`). |
| `nuget-file-path` | The full path to the NuGet file containing the build objects.         |
| `build-status`    | The build/release outcome: Release Created, No Release, or Release Failed. |

---

## Usage

**Important:**

- This action must be run on a runner that supports the `dotnet` CLI (e.g., `windows-latest`, `ubuntu-latest`) and should be triggered by a pull request that has been closed and merged.
- The `actions/setup-dotnet` action must be called before this action to set up the .NET environment. Be sure to specify the required .NET SDK version.
- If your project relies on external NuGet sources, configure them (e.g., by running `dotnet nuget add source`) before using this action.

### Example Workflow

```yaml
name: Build and Release NuGet Package

on:
  pull_request:
    types: [closed]

jobs:
  build-and-release-nuget:
    if: github.event.pull_request.merged == true
    runs-on: windows-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x' # Specify your required .NET SDK version

      - name: Add NuGet Source (if needed)
        run: dotnet nuget add source --username ${{ github.repository_owner }} --password ${{ env.GITHUB_TOKEN }} --store-password-in-clear-text --name nuget.org "https://api.nuget.org/v3/index.json"
        shell: pwsh

      - name: Build .NET Project and Create NuGet Release
        uses: lee-lott-actions/build-vsproject-net-nuget-release@v1
        with:
          project-file-path: 'path/to/your/project.csproj'
          prerelease: 'false'
          prerelease-suffix: 'beta'
          package-name: 'my-nuget-package'
          configuration: 'Release'
          token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Notes

- The action will only proceed if the PR was merged (`github.event.pull_request.merged == true`).
- Tags and releases follow [semantic versioning](https://semver.org/) and support pre-release naming.
- A summary of the release, including the release tag, the NuGet package name, and the release URL, is added to the job summary.
- For pre-releases, an annotated tag is created but a GitHub Release is not published.
- For standard releases, an annotated tag is created, the NuGet package is attached, and a GitHub Release is published with notes and timestamps.

---
