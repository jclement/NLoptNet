# JClement.NLoptNet

A modernized C# wrapper for the [NLopt](https://github.com/stevengj/nlopt) nonlinear optimization library. This package provides pre-built native binaries for NLopt 2.10.1 across all major platforms.

## Why This Package Exists

This is a fork of [BrannonKing/NLoptNet](https://github.com/BrannonKing/NLoptNet) with the following enhancements:

- **Updated to NLopt 2.10.1** - Latest version with bug fixes and improvements
- **macOS Support** - Added support for both Intel (x64) and Apple Silicon (arm64)
- **Modern .NET** - Targets .NET 6.0, .NET 8.0, and .NET Standard 2.0
- **Automated Builds** - All native binaries built from source via CI/CD
- **Cross-Platform** - Comprehensive platform coverage out of the box

Thanks to **Brannon King** for creating the original NLoptNet wrapper and to **ASI** for sponsoring development on the original project. This modernization by **Jeff Clement** builds on that foundation to keep the wrapper current with modern .NET and NLopt releases.

## Platform Support

Includes native binaries for:

- **Windows**: x86 (32-bit), x64 (64-bit)
- **Linux**: x64, arm64, musl-x64 (Alpine Linux)
- **macOS**: x64 (Intel), arm64 (Apple Silicon)

Framework support:

- .NET Standard 2.0+ (compatible with .NET Framework 4.7.1+)
- .NET 6.0+
- .NET 8.0+

## Installation

```bash
dotnet add package JClement.NLoptNet
```

Or via Package Manager:

```powershell
Install-Package JClement.NLoptNet
```

## Quick Start

```csharp
using NLoptNet;

// Create solver: algorithm, dimension, tolerance, max iterations
using (var solver = new NLoptSolver(NLoptAlgorithm.LN_COBYLA, 1, 0.001, 100))
{
    solver.SetLowerBounds(new[] { -10.0 });
    solver.SetUpperBounds(new[] { 100.0 });

    // Minimize f(x) = (x - 3)² + 4
    solver.SetMinObjective(variables =>
    {
        return Math.Pow(variables[0] - 3.0, 2.0) + 4.0;
    });

    var initialValue = new[] { 2.0 };
    var result = solver.Optimize(initialValue, out double? finalScore);

    Console.WriteLine($"Result: {result}");
    Console.WriteLine($"Optimal x: {initialValue[0]}");  // ~3.0
    Console.WriteLine($"Optimal f(x): {finalScore}");     // ~4.0
}
```

## Features

### Derivative-Free Algorithms

- **LN_COBYLA** - Constrained Optimization BY Linear Approximations
- **LN_BOBYQA** - Bound Optimization BY Quadratic Approximation
- **LN_NELDERMEAD** - Nelder-Mead simplex
- **LN_SBPLX** - Subplex variant

### Gradient-Based Algorithms

```csharp
solver.SetMinObjective((variables, gradient) =>
{
    if (gradient != null)
        gradient[0] = (variables[0] - 3.0) * 2.0;
    return Math.Pow(variables[0] - 3.0, 2.0) + 4.0;
});
```

- **LD_LBFGS** - Low-storage BFGS
- **LD_MMA** - Method of Moving Asymptotes
- **LD_SLSQP** - Sequential Least Squares Quadratic Programming

### Global Optimization

- **GN_DIRECT** - DIRECT algorithm
- **GN_CRS2_LM** - Controlled Random Search
- **GN_ISRES** - Improved Stochastic Ranking Evolution Strategy

### Constraints

```csharp
// Equality constraint: g(x) = 0
solver.AddEqualZeroConstraint((variables, gradient) =>
{
    if (gradient != null)
        gradient[0] = 1.0;
    return variables[0] - 100.0;
});

// Inequality constraint: h(x) <= 0
solver.AddLessOrEqualZeroConstraint((variables, gradient) =>
{
    if (gradient != null)
        gradient[0] = 1.0;
    return variables[0] - 50.0;
});
```

### Augmented Lagrangian

```csharp
using(var solver = new NLoptSolver(
    NLoptAlgorithm.LN_AUGLAG,    // Main algorithm
    2,                            // Parameters
    0.001,                        // Tolerance
    1000,                         // Max iterations
    NLoptAlgorithm.LN_SBPLX))     // Sub-algorithm
{
    solver.SetLowerBounds(new[] { 1.0, 1.0 });
    solver.SetUpperBounds(new[] { 200.0, 200.0 });
    solver.AddLessOrEqualZeroConstraint(Constrain);
    solver.SetMinObjective(Score);

    var best = new[] { 10.0, 10.0 };
    var result = solver.Optimize(best, out double? finalScore);
}
```

## Documentation

- [NLopt Documentation](https://nlopt.readthedocs.io/) - Algorithm reference and details
- [NLopt Algorithms](https://nlopt.readthedocs.io/en/latest/NLopt_Algorithms/) - Complete algorithm list
- [GitHub Repository](https://github.com/jclement/NLoptNet) - Source code and examples

## Troubleshooting

### "DllNotFoundException: Unable to load DLL 'nlopt'"

This usually means:

1. NuGet package not restored - try `dotnet restore`
2. Unsupported platform - check Platform Support above
3. Runtime-specific assets not copied - ensure your `.csproj` has proper `<RuntimeIdentifier>` set

For platform-specific builds:

```xml
<PropertyGroup>
  <RuntimeIdentifier>linux-x64</RuntimeIdentifier>
</PropertyGroup>
```

For multi-platform:

```xml
<PropertyGroup>
  <RuntimeIdentifiers>win-x64;linux-x64;osx-arm64</RuntimeIdentifiers>
</PropertyGroup>
```

## Publishing Your Application

Specify the runtime when publishing:

```bash
# Windows x64
dotnet publish -r win-x64 --self-contained

# Linux x64
dotnet publish -r linux-x64 --self-contained

# macOS Apple Silicon
dotnet publish -r osx-arm64 --self-contained
```

## License

This wrapper and NLopt are both licensed under the **LGPL 2.1 or later**.

This means:
- ✅ You can use it in commercial applications
- ✅ Dynamic linking is permitted (which this package does)
- ⚠️ If you modify NLopt itself, you must share those modifications

See [NLopt documentation](https://nlopt.readthedocs.io/) for full license details.

## Credits

- **[Steven G. Johnson](https://math.mit.edu/~stevenj/)** - Original NLopt library author
- **[Brannon King](https://github.com/BrannonKing)** - Original NLoptNet C# wrapper
- **[ASI](http://asirobots.com)** - Sponsored original NLoptNet development
- **[Jeff Clement](https://github.com/jclement)** - Modernization and macOS support

## Contributing

Contributions welcome! Please see the [GitHub repository](https://github.com/jclement/NLoptNet) for details.

For NLopt-specific questions, consult the [NLopt documentation](https://nlopt.readthedocs.io/) before opening issues.
