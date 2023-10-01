>[!tip]
>Uses **csc.exe** compiler (`c:\Users\Administrator\Desktop>C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe`)

Basic C# program:

```dotnet
using System;

namespace dotnetapp
{
	class Program
	{
		static void Main(string[] args)
		{
			Console.WriteLine("What is your favourite Web Application Language?");
			String answer = Console.ReadLine();
			Console.WriteLine("Your answer was: " + answer + "\r\n");
		}
	}
}
```

# Working with dotNET and C\#

## Decompile C\#

- [ILSpy](https://github.com/icsharpcode/ILSpy) (decompiler)
- [dnSpy](../Tools/dnSpy.md) (decompiler and debugger)

## Debugging dotNET

Some optimization may prevent trivial debugging of dotNET application. One of the ways these optimizations manifest themselves in a debugging session is by preventing us from setting breakpoints at arbitrary code lines.

Fortunately, there is a way to modify how a target executable is optimized at runtime [^debug-easy] :

[^debug-easy]: https://github.com/dnSpy/dnSpy/wiki/Making-an-Image-Easier-to-Debug

```dotnet
// change this
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints)]

// with this
[assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default |
DebuggableAttribute.DebuggingModes.DisableOptimizations |
DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints |
DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
```

Once done, *Compile* and save the edited assembly (`File > Save Module`).