# Using NLoptNet as a NuGet Package

## Installation

### Option 1: Visual Studio NuGet Package Manager

1. Right-click on your project in Solution Explorer
2. Select "Manage NuGet Packages"
3. Search for "NLoptNet"
4. Click "Install"

### Option 2: .NET CLI

```bash
dotnet add package NLoptNet --version 2.10.1
```

### Option 3: Package Manager Console

```powershell
Install-Package NLoptNet -Version 2.10.1
```

### Option 4: Edit .csproj directly

Add this to your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="NLoptNet" Version="2.10.1" />
</ItemGroup>
```

## Platform Support

NLoptNet 2.10.1 includes native binaries for:

- **Windows**: x86 (32-bit), x64 (64-bit)
- **Linux**: x64, arm64, musl-x64 (Alpine Linux)
- **macOS**: x64 (Intel), arm64 (Apple Silicon)

The correct native library is automatically selected at runtime based on your platform.

## Framework Support

- .NET Standard 2.0+ (compatible with .NET Framework 4.7.1+)
- .NET 6.0+
- .NET 8.0+

## Basic Usage

```csharp
using NLoptNet;

// Create a solver with 1 parameter, tolerance 0.001, max 100 iterations
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
    Console.WriteLine($"Optimal x: {initialValue[0]}");
    Console.WriteLine($"Optimal f(x): {finalScore}");
}
```

## Advanced Examples

### With Gradient

```csharp
solver.SetMinObjective((variables, gradient) =>
{
    if (gradient != null)
        gradient[0] = (variables[0] - 3.0) * 2.0;
    return Math.Pow(variables[0] - 3.0, 2.0) + 4.0;
});
```

### With Constraints

```csharp
// Add equality constraint: x = 100
solver.AddEqualZeroConstraint((variables, gradient) =>
{
    if (gradient != null)
        gradient[0] = 1.0;
    return variables[0] - 100.0;
});

// Add inequality constraint: x <= 50
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
    NLoptAlgorithm.LN_AUGLAG,  // Main algorithm
    2,                          // Number of parameters
    0.001,                      // Tolerance
    1000,                       // Max iterations
    NLoptAlgorithm.LN_SBPLX))   // Sub-algorithm
{
    solver.SetLowerBounds(new[] { 1.0, 1.0 });
    solver.SetUpperBounds(new[] { 200.0, 200.0 });
    solver.AddLessOrEqualZeroConstraint(Constrain);
    solver.SetMinObjective(Score);

    var best = new[] { 10.0, 10.0 };
    var result = solver.Optimize(best, out double? finalScore);
}
```

## Available Algorithms

NLopt provides many optimization algorithms. Some commonly used ones:

### Derivative-Free
- `LN_COBYLA` - Constrained Optimization BY Linear Approximations
- `LN_BOBYQA` - Bound Optimization BY Quadratic Approximation
- `LN_NELDERMEAD` - Nelder-Mead simplex
- `LN_SBPLX` - Subplex (Nelder-Mead variant)

### Gradient-Based
- `LD_LBFGS` - Low-storage BFGS
- `LD_MMA` - Method of Moving Asymptotes
- `LD_SLSQP` - Sequential Least Squares Quadratic Programming

### Global Optimization
- `GN_DIRECT` - DIRECT algorithm
- `GN_CRS2_LM` - Controlled Random Search
- `GN_ISRES` - Improved Stochastic Ranking Evolution Strategy

See the [NLopt documentation](https://nlopt.readthedocs.io/en/latest/NLopt_Algorithms/) for a complete list.

## Troubleshooting

### "DllNotFoundException: Unable to load DLL 'nlopt'"

This usually means:
1. The NuGet package wasn't restored correctly - try `dotnet restore`
2. You're on an unsupported platform (check Platform Support above)
3. The runtime-specific assets aren't being copied - check your `.csproj` has proper `<RuntimeIdentifier>` or `<RuntimeIdentifiers>` set

### Platform-Specific Builds

If you're building for a specific platform, set the RuntimeIdentifier:

```xml
<PropertyGroup>
  <RuntimeIdentifier>linux-x64</RuntimeIdentifier>
</PropertyGroup>
```

Or for multi-platform support:

```xml
<PropertyGroup>
  <RuntimeIdentifiers>win-x64;linux-x64;osx-arm64</RuntimeIdentifiers>
</PropertyGroup>
```

## Publishing Your Application

When publishing a self-contained application, specify the runtime:

```bash
# Windows x64
dotnet publish -r win-x64 --self-contained

# Linux x64
dotnet publish -r linux-x64 --self-contained

# macOS Apple Silicon
dotnet publish -r osx-arm64 --self-contained
```

## License

NLopt is licensed under the LGPL 2.1 or later. This means:
- You can use NLoptNet in commercial applications
- You can link dynamically (which this package does)
- If you modify NLopt itself, you must share those modifications

See the [NLopt documentation](https://nlopt.readthedocs.io/) for full license details.

## Additional Resources

- [NLopt Documentation](https://nlopt.readthedocs.io/)
- [NLopt GitHub](https://github.com/stevengj/nlopt)
- [NLoptNet GitHub](https://github.com/jclement/NLoptNet)
