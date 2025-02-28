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

- [Advanced .NET Exploitation Training](https://summoning.team/) public course by Summoning Team

## Decompile C\#

- [ILSpy](https://github.com/icsharpcode/ILSpy) (decompiler)
- [dnSpy](../Tools/dnSpy.md) (decompiler and debugger)

## Debugging dotNET

Some optimization may prevent trivial debugging of #dotNET application. One of the ways these optimizations manifest themselves in a debugging session is by preventing us from setting breakpoints at arbitrary code lines.

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

# Known vulnerable components and tricks

## .NET Remoting

[.NET Remoting](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-netod/bfd49902-36d7-4479-bf75-a2431bd99039) is a Microsoft application programming interface (API) for interprocess communication released in 2002 with the 1.0 version of .NET Framework.

- [Breaking .NET Through Serialization](https://media.blackhat.com/bh-us-12/Briefings/Forshaw/BH_US_12_Forshaw_Are_You_My_Type_WP.pdf), BH_US_12_Forshaw_Are_You_My_Type_WP.pdf
- [From Blackbox .NET Remoting to Unauthenticated Remote Code Execution](https://code-white.com/blog/2023-07-from-blackbox-dotnet-remoting-to-rce/), code-white.com
- [Teaching the Old .NET Remoting New Exploitation Tricks](https://code-white.com/blog/teaching-the-old-net-remoting-new-exploitation-tricks/), code-white.com
- [Leaking ObjRefs to Exploit HTTP .NET Remoting](https://code-white.com/blog/leaking-objrefs-to-exploit-http-dotnet-remoting/), code-white.com
