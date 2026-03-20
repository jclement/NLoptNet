# Publishing NLoptNet to NuGet

## Fully Automated Publication

The `build-and-publish.yml` workflow handles everything automatically:

1. **Daily Version Checks** (2am UTC)
   - Detects new NLopt releases from GitHub
   - Automatically builds and publishes if new version found

2. **Complete Build Process**
   - Builds NLopt from source for all platforms
   - Creates NuGet package with all native binaries
   - Tests on Windows, Linux, and macOS
   - Publishes to NuGet.org
   - Creates GitHub release with git tag

## One-Time Setup

### Configure NuGet Publishing

1. **Get NuGet API Key**
   - Go to https://www.nuget.org/account/apikeys
   - Create new API key with "Push" permission
   - Select packages: `NLoptNet`
   - Set expiration (or no expiration)

2. **Add to GitHub Secrets**
   - Go to https://github.com/jclement/NLoptNet/settings/secrets/actions
   - Click "New repository secret"
   - Name: `NUGET_API_KEY`
   - Value: Your API key from step 1
   - Click "Add secret"

**That's it!** The workflow is now fully automated.

## How It Works

### Automatic Publication (Daily)

Every day at 2am UTC, the workflow:
1. Checks for new NLopt releases
2. If new version found:
   - Builds native binaries for all platforms
   - Updates package version automatically
   - Builds NuGet package
   - Runs tests on all platforms
   - Publishes to NuGet.org
   - Creates GitHub release with tag `v{version}`

### Manual Trigger

To manually build and publish a specific version:

1. Go to https://github.com/jclement/NLoptNet/actions/workflows/build-and-publish.yml
2. Click "Run workflow"
3. Enter NLopt version (e.g., `2.10.1`) or leave as `latest`
4. Click "Run workflow"

The workflow will:
- Build that specific version
- Run all tests
- Publish to NuGet.org (if version doesn't already exist)
- Create GitHub release

## What Gets Published

### NuGet.org Package

Package: https://www.nuget.org/packages/NLoptNet/

Installation:
```bash
dotnet add package NLoptNet --version {version}
```

Includes native binaries for:
- Windows x86, x64
- Linux x64, arm64, musl-x64 (Alpine)
- macOS x64 (Intel), arm64 (Apple Silicon)

### GitHub Release

Release: https://github.com/jclement/NLoptNet/releases

Each release includes:
- Git tag `v{version}`
- NuGet package (.nupkg file)
- Release notes with installation instructions

## Version Management

The workflow automatically:
- Detects NLopt version from upstream repository
- Updates `.csproj` file with new version
- Creates git tag for that version
- Only publishes if version is new (won't re-publish existing versions)

## Monitoring

### Check Workflow Status

View all workflow runs:
https://github.com/jclement/NLoptNet/actions/workflows/build-and-publish.yml

### Workflow Notifications

GitHub will notify you:
- Email on workflow failures
- Can configure Slack/Teams/Discord notifications in workflow

### NuGet Publication Status

After workflow completes:
1. Check NuGet.org: https://www.nuget.org/packages/NLoptNet/
2. May take 15-30 minutes for indexing
3. Test installation:
   ```bash
   dotnet new console -n test
   cd test
   dotnet add package NLoptNet --version {version}
   ```

## Testing Before Publication

The workflow includes automatic testing on all platforms:

1. **Build Test** - Verifies package builds correctly
2. **Integration Test** - Creates test project, adds package, runs optimization
3. **Platform Test** - Runs on Ubuntu, Windows, macOS runners

Tests must pass before publication to NuGet.

To test locally before triggering workflow:

```bash
# Clone repo
git clone https://github.com/jclement/NLoptNet.git
cd NLoptNet

# Manually run the workflow or test specific platform
# (The workflow handles cross-compilation, local testing is limited to your platform)
```

## Troubleshooting

### Workflow Fails on Build

**Symptom:** Build job fails on specific platform

**Solutions:**
- Check CMake configuration for that platform
- Verify cross-compilation tools are available
- Check NLopt source for build issues (may need patches)

### Tests Fail

**Symptom:** Test job fails on specific platform

**Solutions:**
- Library may not be loading correctly
- Check DllImport names match native library names
- Verify runtime-specific paths are correct

### Publication Fails

**Symptom:** Publish job fails with authentication error

**Solutions:**
- Check `NUGET_API_KEY` secret is set correctly
- Verify API key has "Push" permission
- Check API key hasn't expired

**Symptom:** "Version already exists" error

**Solutions:**
- This is normal - workflow detects existing versions and skips
- To force rebuild, manually increment version and trigger workflow

### Package Missing Native Binaries

**Symptom:** Package installs but fails to load native library

**Solutions:**
- Check `.csproj` includes all `<None Include="runtimes/...">` entries
- Verify artifacts were downloaded correctly in workflow
- Check `PackagePath` matches expected structure

## Manual Publishing (Not Recommended)

If you need to publish manually for some reason:

```bash
# Build package locally (requires all native binaries)
dotnet pack NLoptNet/NLoptNet.csproj --configuration Release --output ./pack

# Publish to NuGet.org
dotnet nuget push ./pack/NLoptNet.{version}.nupkg \
  --api-key YOUR_API_KEY \
  --source https://api.nuget.org/v3/index.json
```

**Note:** This requires you to manually build all native binaries first. The automated workflow is strongly recommended.

## Updating Workflow

To modify the build/publish process:

1. Edit `.github/workflows/build-and-publish.yml`
2. Test changes on a branch
3. Merge to `main` when ready

The workflow file itself is version-controlled, so changes are tracked.

## Support

- **Workflow Issues:** https://github.com/jclement/NLoptNet/issues
- **NLopt Issues:** https://github.com/stevengj/nlopt/issues
- **NuGet Issues:** https://github.com/NuGet/Home/issues

## Summary

With the `NUGET_API_KEY` secret configured, the entire process is fully automated:

✅ Daily checks for new NLopt versions
✅ Automatic builds for all platforms
✅ Automatic testing
✅ Automatic publication to NuGet.org
✅ Automatic GitHub releases with tags

**You don't need to do anything.** New NLopt versions will automatically become available as NuGet packages within a few hours of release.
