# NLOptNet Modernization Plan

## Goal
Update NLOptNet from NLopt 2.6.1 to 2.10.1 and add macOS support (both x64 and arm64).

## Current State
- **NLopt Version**: 2.6.1 (from 2020)
- **Platforms**: Windows (x86/x64), Linux (x64, arm64, musl-x64)
- **Missing**: macOS support
- **Target Frameworks**: netstandard2.0, net47
- **Last Updated**: October 2022

## Target State
- **NLopt Version**: 2.10.1 (March 2026)
- **Platforms**: Windows (x86/x64), Linux (x64, arm64, musl-x64), **macOS (x64, arm64)**
- **Target Frameworks**: netstandard2.0, net6.0, net8.0
- **Package**: Modern NuGet with proper runtime-specific assets

## Sources for Binaries

### Option 1: Extract from Python Wheels (Easiest)
Python's nlopt package (v2.10.0) has prebuilt wheels for all platforms:
- https://pypi.org/project/nlopt/#files
- Platforms: Windows (AMD64, x86), Linux (x86_64, aarch64, etc.), macOS (x86_64, arm64)
- These contain compiled .so/.dll/.dylib files we can extract

### Option 2: Build from Source (Most Control)
- Source: https://github.com/stevengj/nlopt/releases/tag/v2.10.1
- Build system: CMake
- Requires: C compiler, CMake 3.15+
- Cross-compilation needed for all platforms

### Option 3: Use conda-forge binaries
- conda-forge has NLopt 2.10.1 for all platforms
- Can extract from conda packages

##Implementation Steps

### 1. Obtain Binaries

**Recommended Approach: Extract from Python Wheels**

```bash
# Download wheels for all platforms
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-win_amd64.whl
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-win32.whl
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-manylinux_2_17_x86_64.whl
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-manylinux_2_17_aarch64.whl
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-macosx_10_13_x86_64.whl
wget https://files.pythonhosted.org/packages/.../nlopt-2.10.0-cp313-cp313-macosx_11_0_arm64.whl

# Extract (wheels are just zip files)
unzip nlopt-*.whl
# Native libraries will be in nlopt.libs/ or nlopt/ directory
```

### 2. Update Project Structure

Add to `NLoptNet.csproj`:

```xml
<!-- macOS x64 -->
<None Include="runtimes\osx-x64\native\libnlopt.dylib" Pack="true" PackagePath="runtimes/osx-x64/native/libnlopt.dylib">
  <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>

<!-- macOS arm64 -->
<None Include="runtimes\osx-arm64\native\libnlopt.dylib" Pack="true" PackagePath="runtimes/osx-arm64/native/libnlopt.dylib">
  <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>
```

Update RuntimeIdentifiers:
```xml
<RuntimeIdentifiers>linux-arm64;linux-musl-x64;linux-x64;osx-x64;osx-arm64;win-x64;win-x86</RuntimeIdentifiers>
```

### 3. Update P/Invoke Code

Check `NLoptInterop.cs` for platform-specific DLL loading:

```csharp
private const string LibraryName = "nlopt";  // Works across platforms with runtime-specific paths

// Windows: nlopt.dll
// Linux: nlopt.so or libnlopt.so
// macOS: libnlopt.dylib
```

.NET's runtime-specific package resolution should handle this automatically.

### 4. Update Package Metadata

```xml
<PackageVersion>2.10.1</PackageVersion>
<AssemblyVersion>2.10.1.0</AssemblyVersion>
<FileVersion>2.10.1.0</FileVersion>
<Authors>Brannon King, Jeff Clement</Authors>
<Copyright>2026</Copyright>
<PackageReleaseNotes>Updated to NLopt 2.10.1, added macOS support (x64 and arm64)</PackageReleaseNotes>
<RepositoryUrl>https://github.com/jclement/NLoptNet</RepositoryUrl>
```

Add modern target frameworks:
```xml
<TargetFrameworks>netstandard2.0;net6.0;net8.0</TargetFrameworks>
```

### 5. Create Build/Download Script

Create `scripts/download-nlopt-binaries.sh`:

```bash
#!/bin/bash
set -e

NLOPT_VERSION="2.10.0"
DOWNLOAD_DIR="downloads"
RUNTIMES_DIR="NLoptNet/runtimes"

mkdir -p "$DOWNLOAD_DIR"

# Download Python wheels
echo "Downloading NLopt $NLOPT_VERSION wheels..."

# Function to download and extract
download_and_extract() {
    local url=$1
    local platform=$2
    local lib_name=$3
    local dest_dir=$4

    echo "Processing $platform..."
    wget -q "$url" -O "$DOWNLOAD_DIR/temp.whl"
    unzip -q "$DOWNLOAD_DIR/temp.whl" -d "$DOWNLOAD_DIR/temp"

    mkdir -p "$dest_dir"
    find "$DOWNLOAD_DIR/temp" -name "$lib_name" -exec cp {} "$dest_dir/" \;

    rm -rf "$DOWNLOAD_DIR/temp" "$DOWNLOAD_DIR/temp.whl"
}

# Windows x64
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-win_amd64.whl" \
    "win-x64" \
    "nlopt.dll" \
    "$RUNTIMES_DIR/win-x64/native"

# Windows x86
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-win32.whl" \
    "win-x86" \
    "nlopt.dll" \
    "$RUNTIMES_DIR/win-x86/native"

# Linux x64
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-manylinux_2_17_x86_64.whl" \
    "linux-x64" \
    "*.so*" \
    "$RUNTIMES_DIR/linux-x64/native"

# Linux arm64
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-manylinux_2_17_aarch64.whl" \
    "linux-arm64" \
    "*.so*" \
    "$RUNTIMES_DIR/linux-arm64/native"

# macOS x64
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-macosx_10_13_x86_64.whl" \
    "osx-x64" \
    "*.dylib" \
    "$RUNTIMES_DIR/osx-x64/native"

# macOS arm64
download_and_extract \
    "https://files.pythonhosted.org/packages/.../nlopt-${NLOPT_VERSION}-cp313-cp313-macosx_11_0_arm64.whl" \
    "osx-arm64" \
    "*.dylib" \
    "$RUNTIMES_DIR/osx-arm64/native"

echo "All binaries downloaded and extracted"
```

### 6. Update GitHub Actions

Create `.github/workflows/build.yml` to:
- Build for all target frameworks
- Run tests on Windows, Linux, macOS
- Pack NuGet package
- Publish to NuGet on release

### 7. Testing

Add test projects for each platform if not already present:
- Test on Windows (x64, x86)
- Test on Linux (x64, arm64 via Docker/QEMU)
- Test on macOS (x64, arm64)

## Challenges

1. **Library naming**: Windows uses `nlopt.dll`, Linux uses `libnlopt.so`, macOS uses `libnlopt.dylib`
   - Solution: .NET's DllImport handles this via runtime-specific paths

2. **License compliance**: NLopt is LGPL
   - Solution: Document clearly, include LICENSE file, note in package description

3. **Binary size**: Including binaries for all platforms increases package size
   - Solution: This is standard for native NuGet packages - unavoidable

4. **Testing on all platforms**: Need access to Windows, Linux, macOS
   - Solution: Use GitHub Actions with matrix builds

## Timeline

1. **Phase 1**: Get all binaries (1-2 hours)
   - Download Python wheels
   - Extract native libraries
   - Organize in runtimes/ structure

2. **Phase 2**: Update project (30 mins)
   - Update csproj with new platforms
   - Update package metadata
   - Test builds locally

3. **Phase 3**: Testing (1-2 hours)
   - Test on available platforms
   - Set up GitHub Actions
   - Validate all platforms load correctly

4. **Phase 4**: Documentation & Release (30 mins)
   - Update README
   - Create release notes
   - Publish to NuGet

## Next Steps

1. Get exact URLs for Python wheel files from PyPI
2. Download and extract binaries
3. Update project files
4. Test locally
5. Push to GitHub
6. Set up CI/CD
7. Publish package

## Resources

- [NLopt GitHub](https://github.com/stevengj/nlopt)
- [NLopt Documentation](https://nlopt.readthedocs.io/)
- [PyPI NLopt](https://pypi.org/project/nlopt/)
- [NuGet Runtime Package Guidelines](https://docs.microsoft.com/en-us/nuget/create-packages/supporting-multiple-target-frameworks)
