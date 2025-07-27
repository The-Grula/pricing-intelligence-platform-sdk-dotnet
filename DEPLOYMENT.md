# Deployment Instructions

This document contains instructions for setting up automated NuGet publishing for the Grula Pricing Intelligence Platform SDK.

## Prerequisites

1. **NuGet Account**: You need a NuGet.org account
2. **API Key**: Generate an API key from your NuGet.org account
3. **GitHub Repository**: The code should be pushed to the GitHub repository

## Setup Instructions

### 1. Generate NuGet API Key

1. Go to [NuGet.org](https://www.nuget.org/)
2. Sign in to your account
3. Go to your account settings
4. Navigate to "API Keys"
5. Click "Create" to generate a new API key
6. Set the following permissions:
   - **Package Owner**: Select your account
   - **Scopes**: Push new packages and package versions
   - **Packages**: All packages (or specify `Grula.PricingIntelligencePlatform.Sdk`)
   - **Expiration**: Set appropriate expiration date

### 2. Add API Key to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Name: `NUGET_API_KEY`
5. Value: Paste your NuGet API key
6. Click "Add secret"

### 3. Publishing Process

#### Automatic Publishing (Recommended)

1. **Create a Git Tag**: 
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. **GitHub Actions will automatically**:
   - Build the project
   - Create NuGet packages
   - Publish to NuGet.org

#### Manual Publishing

You can also trigger the workflow manually:

1. Go to GitHub repository → Actions
2. Select "Build & Publish" workflow
3. Click "Run workflow"
4. Choose the branch and click "Run workflow"

### 4. Version Management

- **Release versions**: Use tags like `v1.0.0`, `v1.1.0`, `v2.0.0`
- **Preview versions**: Manual workflow runs create `0.1.0-preview` packages
- **Version format**: Follow semantic versioning (SemVer)

### 5. Verification

After publishing:

1. Check the GitHub Actions workflow completed successfully
2. Verify the package appears on [NuGet.org](https://www.nuget.org/packages/Grula.PricingIntelligencePlatform.Sdk/)
3. Test installation: `dotnet add package Grula.PricingIntelligencePlatform.Sdk`

## Troubleshooting

### Common Issues

1. **Build Failures**:
   - Check that the OpenAPI endpoint is accessible
   - Verify all dependencies are correctly referenced

2. **Publishing Failures**:
   - Verify the `NUGET_API_KEY` secret is set correctly
   - Check API key permissions and expiration
   - Ensure package version doesn't already exist

3. **Version Conflicts**:
   - Each version can only be published once
   - Use different version numbers for different releases

### Workflow Status

Monitor the workflow status at:
`https://github.com/The-Grula/pricing-intelligence-platform-sdk-dotnet/actions`

## Local Development

### Building Locally

```bash
dotnet restore src/PricingIntelligencePlatform.sln
dotnet build src/PricingIntelligencePlatform.sln --configuration Release
```

### Creating Local Package

```bash
dotnet pack src/PricingIntelligencePlatform.Sdk/PricingIntelligencePlatform.Sdk.csproj --configuration Release --output ./nupkg
```

### Testing Local Package

```bash
dotnet new nugetconfig
dotnet nuget add source ./nupkg --name local
dotnet add package Grula.PricingIntelligencePlatform.Sdk --source local
```

## Package Information

- **Package ID**: `Grula.PricingIntelligencePlatform.Sdk`
- **Target Framework**: .NET 8.0
- **License**: MIT
- **Repository**: https://github.com/The-Grula/pricing-intelligence-platform-sdk-dotnet
- **API Documentation**: https://api.grula.net

## Support

For deployment issues:
1. Check GitHub Actions logs
2. Review NuGet.org package status
3. Create an issue in the GitHub repository
