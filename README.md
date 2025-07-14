# Reusable Workflows

This repository contains reusable GitHub Actions workflows for .NET projects. These workflows provide standardized CI/CD processes that can be shared across multiple repositories.

## Available Workflows

### 1. Build, Test and Sonar Analysis (`dotnet-build-test.yml`)

This workflow builds .NET projects, runs tests, and performs SonarCloud analysis.

**Features:**
- Automatic detection of solution (.sln), project (.csproj), and NuSpec (.nuspec) files
- MSBuild compilation with Release configuration
- NuGet package restoration and caching
- VSTest execution for all test assemblies
- SonarCloud static code analysis
- Artifact upload (NuGet packages for ClassLibrary projects, release artifacts for others)

**Inputs:**
- `projectType` (required): Type of project - "ClassLibrary" or "WindowsForm" (default: "WindowsForm")

**Requirements:**
- Solution must contain a .sln file
- At least one .csproj file (excluding test projects)
- A .nuspec file in the same directory as the main .csproj
- Test projects should follow naming convention `*.Test(s).csproj`

**Required Secrets:**
- `SONAR_ORGANIZATION`: Your SonarCloud organization key
- `SONAR_URL`: SonarCloud URL (usually https://sonarcloud.io)
- `SONAR_TOKEN`: SonarCloud authentication token

### 2. NuGet Package Publishing (`dotnet-publish-nuget.yml`)

This workflow publishes NuGet packages to GitHub Packages.

**Features:**
- Downloads artifacts from previous workflow runs
- Configures GitHub Packages as NuGet source
- Publishes packages with duplicate skipping

**Requirements:**
- Must run after the build workflow that produces `nupkg` artifacts
- Requires GitHub token with package write permissions

### 3. Azure Blob Storage Publishing (`publish-to-file-server.yml`)

This workflow publishes release artifacts as ZIP files to Azure Blob Storage.

**Features:**
- Downloads release artifacts from previous workflow runs
- Automatically detects project name from executable files
- Creates ZIP archive of all release files
- Uploads to Azure Blob Storage with organized naming

**Required Secrets:**
- `AZURE_CREDENTIALS`: Azure service principal credentials (JSON format)
- `AZURE_STORAGE_CONNECTION_STRING`: Azure Storage account connection string

## Usage Examples

### Example 1: Class Library with NuGet Publishing

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    uses: SampleProjectDotNet/reusable-workflows/.github/workflows/dotnet-build-test.yml@main
    with:
      projectType: "ClassLibrary"
    secrets:
      SONAR_ORGANIZATION: ${{ secrets.SONAR_ORGANIZATION }}
      SONAR_URL: ${{ secrets.SONAR_URL }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  publish-nuget:
    needs: build-test
    if: github.ref == 'refs/heads/main'
    uses: SampleProjectDotNet/reusable-workflows/.github/workflows/dotnet-publish-nuget.yml@main
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Example 2: Windows Forms Application with Azure Publishing

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    uses: SampleProjectDotNet/reusable-workflows/.github/workflows/dotnet-build-test.yml@main
    with:
      projectType: "WindowsForm"
    secrets:
      SONAR_ORGANIZATION: ${{ secrets.SONAR_ORGANIZATION }}
      SONAR_URL: ${{ secrets.SONAR_URL }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  publish-azure:
    needs: build-test
    if: github.ref == 'refs/heads/main'
    uses: SampleProjectDotNet/reusable-workflows/.github/workflows/publish-to-file-server.yml@main
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
      AZURE_STORAGE_CONNECTION_STRING: ${{ secrets.AZURE_STORAGE_CONNECTION_STRING }}
```

## Setup Instructions

### Prerequisites

1. **SonarCloud Setup:**
   - Create a SonarCloud account at https://sonarcloud.io
   - Create a new project for your repository
   - Generate an authentication token
   - Add the following secrets to your repository:
     - `SONAR_ORGANIZATION`: Your organization key
     - `SONAR_URL`: https://sonarcloud.io
     - `SONAR_TOKEN`: Your authentication token

2. **GitHub Packages Setup (for NuGet publishing):**
   - No additional setup required
   - Uses the repository's `GITHUB_TOKEN` automatically

3. **Azure Blob Storage Setup (for file publishing):**
   - Create an Azure Storage Account
   - Create a service principal with access to the storage account
   - Add the following secrets to your repository:
     - `AZURE_CREDENTIALS`: Service principal JSON
     - `AZURE_STORAGE_CONNECTION_STRING`: Storage account connection string

### Project Structure Requirements

Your .NET project should follow this structure:

```
YourProject/
├── YourProject.sln
├── YourProject/
│   ├── YourProject.csproj
│   ├── YourProject.nuspec  (for ClassLibrary projects)
│   └── (source files)
├── YourProject.Tests/
│   ├── YourProject.Tests.csproj
│   └── (test files)
└── (other projects/folders)
```

## Contributing

Feel free to submit issues and enhancement requests to improve these reusable workflows.

## License

This project is licensed under the MIT License - see the repository license for details.