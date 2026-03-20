# Publishing NLoptNet to NuGet

## Prerequisites

1. **GitHub Repository Setup**
   - Fork is at: https://github.com/jclement/NLoptNet
   - Branch `update-nlopt-2.10.1` has all modernization changes

2. **NuGet.org Account**
   - Create account at https://www.nuget.org/
   - Generate API key at https://www.nuget.org/account/apikeys
   - Add as GitHub secret: `NUGET_API_KEY`

3. **GitHub Secrets Setup**
   - Go to: https://github.com/jclement/NLoptNet/settings/secrets/actions
   - Add secret: `NUGET_API_KEY` with your NuGet API key

## Step-by-Step Publishing Process

### Step 1: Build Native Libraries

The first workflow builds NLopt 2.10.1 from source for all platforms.

1. Go to: https://github.com/jclement/NLoptNet/actions/workflows/build-nlopt.yml
2. Click "Run workflow"
3. Select branch: `update-nlopt-2.10.1`
4. Click "Run workflow"

This will:
- Build NLopt for Windows (x86, x64)
- Build NLopt for Linux (x64, arm64, musl-x64)
- Build NLopt for macOS (x64, arm64)
- Upload artifacts to the workflow run
- Create `runtimes/` folder structure

**Expected output:**
- Artifacts for each platform in the workflow run
- All binaries combined in `nlopt-all-platforms` artifact

### Step 2: Download and Commit Binaries

1. Download the `nlopt-all-platforms` artifact from the workflow run
2. Extract it to `/workspace/group/NLoptNet/NLoptNet/runtimes/`
3. Verify the structure:

```
NLoptNet/runtimes/
├── linux-arm64/native/nlopt.so
├── linux-musl-x64/native/nlopt.so
├── linux-x64/native/nlopt.so
├── osx-arm64/native/libnlopt.dylib
├── osx-x64/native/libnlopt.dylib
├── win-x64/native/nlopt.dll
└── win-x86/native/nlopt.dll
```

4. Commit and push:

```bash
cd /workspace/group/NLoptNet
git add NLoptNet/runtimes/
git commit -m "Add NLopt 2.10.1 native binaries for all platforms

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
git push
```

### Step 3: Build and Test NuGet Package Locally

```bash
cd /workspace/group/NLoptNet

# Restore dependencies
dotnet restore NLoptNet/NLoptNet.csproj

# Build
dotnet build NLoptNet/NLoptNet.csproj --configuration Release

# Pack
dotnet pack NLoptNet/NLoptNet.csproj --configuration Release --output ./pack

# Verify package contents
unzip -l pack/NLoptNet.2.10.1.nupkg | grep runtimes
```

You should see all runtime-specific binaries listed.

### Step 4: Merge to Main Branch

1. Create pull request from `update-nlopt-2.10.1` to `main`
2. Review changes
3. Merge the PR

### Step 5: Create Release Tag

```bash
cd /workspace/group/NLoptNet
git checkout main
git pull

# Create and push tag
git tag -a v2.10.1 -m "NLoptNet 2.10.1 - Updated to NLopt 2.10.1 with macOS support"
git push origin v2.10.1
```

### Step 6: Automatic Publishing

Once you push the `v2.10.1` tag, the `build-package.yml` workflow automatically:
1. Builds the NuGet package
2. Publishes to NuGet.org (using `NUGET_API_KEY` secret)
3. Publishes to GitHub Packages

Monitor at: https://github.com/jclement/NLoptNet/actions

### Step 7: Verify Publication

1. Check NuGet.org: https://www.nuget.org/packages/NLoptNet/
2. Wait for indexing (can take 15-30 minutes)
3. Test installation:

```bash
dotnet new console -n NLoptTest
cd NLoptTest
dotnet add package NLoptNet --version 2.10.1
```

## Alternative: Manual Publishing

If you prefer to publish manually:

```bash
cd /workspace/group/NLoptNet

# Build package
dotnet pack NLoptNet/NLoptNet.csproj --configuration Release --output ./pack

# Publish to NuGet.org
dotnet nuget push ./pack/NLoptNet.2.10.1.nupkg \
  --api-key YOUR_API_KEY \
  --source https://api.nuget.org/v3/index.json
```

## Automated Version Updates

The `build-nlopt.yml` workflow runs daily (2am UTC) to check for new NLopt releases.

If a new version is found:
1. It builds binaries automatically
2. Creates a GitHub release with artifacts
3. You can then update the version in `.csproj` and publish

To manually trigger version check:
1. Go to: https://github.com/jclement/NLoptNet/actions/workflows/build-nlopt.yml
2. Click "Run workflow"

## Package Maintenance

### Updating to a New NLopt Version

When a new NLopt version is released (e.g., 2.10.2):

1. Update `NLOPT_VERSION` in `.github/workflows/build-nlopt.yml`:
   ```yaml
   env:
     NLOPT_VERSION: '2.10.2'
   ```

2. Update version in `NLoptNet/NLoptNet.csproj`:
   ```xml
   <PackageVersion>2.10.2</PackageVersion>
   <AssemblyVersion>2.10.2.0</AssemblyVersion>
   <FileVersion>2.10.2.0</FileVersion>
   ```

3. Run build workflow to generate new binaries
4. Download and commit binaries
5. Create new tag and push

### Testing Before Release

Always test the package locally before publishing:

```bash
# Build local package
dotnet pack --configuration Release --output ./pack

# Test in a separate project
dotnet new console -n TestProject
cd TestProject
dotnet add package NLoptNet --source ../NLoptNet/pack
```

## Troubleshooting

### Build Failures

**Windows build fails:**
- Check CMake is installed on Windows runner
- Verify architecture flags (`x64` vs `Win32`)

**Linux cross-compilation fails:**
- Ensure cross-compilation tools are installed
- Check `gcc-aarch64-linux-gnu` for arm64 builds
- Check `musl-tools` for musl builds

**macOS build fails:**
- Verify `CMAKE_OSX_ARCHITECTURES` is set correctly
- Check Xcode Command Line Tools are available

### Publishing Failures

**NuGet.org rejects package:**
- Version already exists (can't overwrite)
- Invalid package metadata
- Missing required fields

**Package doesn't include binaries:**
- Check `<None Include="runtimes/...">` entries in `.csproj`
- Verify files exist in repository
- Check `Pack="true"` is set

## Support

- Issues: https://github.com/jclement/NLoptNet/issues
- NLopt Documentation: https://nlopt.readthedocs.io/
- NuGet Package Page: https://www.nuget.org/packages/NLoptNet/
