NLoptNet
========

This is a C# wrapper around the NLopt C library. It includes native binaries for NLopt 2.10.1 across all major platforms:

- **Windows**: x86, x64
- **Linux**: x64, arm64, musl-x64 (Alpine)
- **macOS**: x64 (Intel), arm64 (Apple Silicon)

This package inherits NLopt's LGPL license.

Originally created by [Brannon King](https://github.com/BrannonKing/NLoptNet). Thanks to [ASI](http://asirobots.com) for sponsoring some time on the original project. Modernization and macOS support by [Jeff Clement](https://github.com/jclement).

Example:
```csharp
[TestMethod]
public void FindParabolaMinimum()
{
	using (var solver = new NLoptSolver(NLoptAlgorithm.LN_COBYLA, 1, 0.001, 100))
	{
		solver.SetLowerBounds(new[] { -10.0 });
		solver.SetUpperBounds(new[] { 100.0 });
				
		solver.SetMinObjective(variables =>
		{
			return Math.Pow(variables[0] - 3.0, 2.0) + 4.0;
		});
		double? finalScore;
		var initialValue = new[] { 2.0 };
		var result = solver.Optimize(initialValue, out finalScore);

		Assert.AreEqual(NloptResult.XTOL_REACHED, result);
		Assert.AreEqual(3.0, initialValue[0], 0.1);
		Assert.AreEqual(4.0, finalScore.Value, 0.1);
	}
}
```
You can use the derivative-based optimizers like this:
```csharp
solver.SetMinObjective((variables, gradient) =>
{
	if (gradient != null)
		gradient[0] = (variables[0] - 3.0) * 2.0;
	return Math.Pow(variables[0] - 3.0, 2.0) + 4.0;
});
```
And add constraints like this:
```csharp
solver.AddEqualZeroConstraint((variables, gradient) =>
{
	if (gradient != null)
		gradient[0] = 1.0;
	return variables[0] - 100.0;
});
```
To use the Augmented Lagrangian:
```csharp
using(var solver = new NLoptSolver(NLoptAlgorithm.LN_AUGLAG, 2, 0.001, 1000, NLoptAlgorithm.LN_SBPLX))
{
	solver.SetLowerBounds(new[] { 1.0, 1.0 });
	solver.SetUpperBounds(new[] { 200.0, 200.0 });
	solver.AddLessOrEqualZeroConstraint(Constrain);
	solver.SetMinObjective(Score);
	var result = solver.Optimize(best, out finalScore);
}
```
## Installation

Install via NuGet Package Manager:

```
Install-Package NLoptNet
```

Or via .NET CLI:

```bash
dotnet add package NLoptNet
```

Or add to your `.csproj` file:

```xml
<PackageReference Include="NLoptNet" Version="2.10.1" />
```

## Platform Support

This package supports:
- .NET Standard 2.0+
- .NET 6.0+
- .NET 8.0+
- .NET Framework 4.7+

Native binaries are automatically selected for your platform at runtime.

## Building from Source

1. Clone the repository
2. Run the GitHub Actions workflow to build NLopt binaries for all platforms
3. Run `dotnet build` to build the managed wrapper
4. Run `dotnet pack` to create the NuGet package

## License

This wrapper is licensed under the same LGPL 2.1 or later license as NLopt itself. See the [NLopt documentation](https://nlopt.readthedocs.io/) for more details.

## Contributing

Contributions are welcome! Please read through the [NLopt documentation](https://nlopt.readthedocs.io/) before posting questions/issues here.
