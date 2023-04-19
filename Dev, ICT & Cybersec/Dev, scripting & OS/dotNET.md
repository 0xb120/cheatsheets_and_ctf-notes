>[!tip]
>Uses **csc.exe** compiler (`c:\Users\Administrator\Desktop>C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe`)

Basic C# program:

```scala
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
