# NLOptNet Modernization Status

## What I've Done

1. ✅ Forked NLOptNet to https://github.com/jclement/NLoptNet
2. ✅ Created feature branch `update-nlopt-2.10.1`
3. ✅ Created comprehensive modernization plan
4. ✅ Investigated obtaining NLopt 2.10.x binaries

## The Challenge

Python wheels from PyPI contain Python extension modules (`_nlopt.pyd`, `_nlopt.so`), not the standalone C library (`nlopt.dll`, `libnlopt.so`, `libnlopt.dylib`) that .NET P/Invoke requires.

The container environment lacks:
- pip (for downloading packages)
- unzip (for extracting archives)
- cmake/gcc/make (for building from source)
- wget (though curl works)

## Two Paths Forward

### Path 1: Build NLopt from Source (Recommended for 2.10.1)

This requires a build environment with CMake and a C compiler. Best done via GitHub Actions:

1. Create `.github/workflows/build-nlopt.yml` that:
   - Checks out NLopt 2.10.1 source
   - Builds for Windows (x64, x86), Linux (x64, arm64, musl), macOS (x64, arm64)
   - Uploads artifacts as GitHub release assets

2. Download those artifacts and add to `runtimes/` folder

3. Update NuGet package

**Benefit**: Gets us NLopt 2.10.1 + full platform support
**Effort**: 2-3 hours to set up proper cross-compilation CI/CD

### Path 2: Incremental Update (Fast Win)

Keep current NLopt 2.6.1 binaries but add macOS support:

1. Extract existing `nlopt.dll` from Windows x64 Python wheel for NLopt 2.6.1
2. Extract existing `libnlopt.so` from Linux Python wheel for NLopt 2.6.1
3. Build or obtain `libnlopt.dylib` for macOS (both architectures)
4. Update project to include macOS runtime identifiers
5. Publish as v1.5.0 with "Added macOS support"
6. Defer NLopt 2.10.1 upgrade to separate effort

**Benefit**: Quick macOS support fix
**Effort**: 30 minutes once you have macOS binaries

## Recommended Next Steps

### If You Want Full 2.10.1 Update:

1. Set up GitHub Actions workflow to build NLopt from source
2. Use cross-compilation or build matrices for all platforms
3. This is the "right" way but requires build infrastructure

### If You Want Quick macOS Fix:

1. On a Mac, install NLopt via brew: `brew install nlopt`
2. Copy `/opt/homebrew/lib/libnlopt.dylib` (arm64) and `/usr/local/lib/libnlopt.dylib` (x64)
3. Add to `runtimes/osx-{arm64,x64}/native/`
4. Update .csproj
5. Publish v1.5.0

### Alternative: Use Conda Packages

Conda-forge has prebuilt NLopt 2.10.1 libraries (not Python wrappers):
```bash
# On a machine with conda:
conda create -n nlopt-extract nlopt=2.10.1
# Then extract the actual .dll/.so/.dylib files from the conda environment
```

## What's Ready Now

- Fork created and branch ready
- Project structure understood
- Modernization plan documented
- You have the roadmap

## What You Need From Me

Decision on which path to take:
1. **Full rebuild with CI/CD** (proper but slower)
2. **Quick macOS-only update** (fast partial win)
3. **Conda extraction approach** (middle ground if you have conda)

Let me know and I'll execute whichever path you prefer!
