Another interesting attack vector that can lead to privilege escalation on Windows operating systems revolves around **unquoted service paths**.
We can use this attack when we have write permissions to a service’s main directory and sub-directories but cannot replace files within them.

As we have seen in the previous section, each Windows service maps to an executable file that will be run when the service is started.
Most of the time, services that accompany third party software are stored under the `C:\Program Files` directory, which contains a space character in its name.
This can potentially be turned into an opportunity for a privilege escalation attack.

When using file or directory paths that contain spaces, the **developers should always ensure that they are enclosed by quotation marks**.
This ensures that they are explicitly declared. However, when that is not the case and a path name is unquoted, it is open to interpretation.
Specifically, in the case of executable paths, **anything that comes after each white-space character will be treated as a potential argument** or option for the executable.

Imagine that we have a service stored in a path such as `C:\Program Files\My Program\My Service\service.exe`
If the service path is stored unquoted, Windows will try to start the following list of programs:
1. `C:\Program.exe`
2. `C:\Program Files\My.exe`
3. `C:\Program Files\My Program\My.exe`
4. `C:\Program Files\My Program\My service\service.exe`

We could name our executable `Program.exe` and place it in `C:\` , or name it `My.exe` and place it in `C:\Program Files`.
However, this would require some unlikely write permissions since standard users do not have write access to these directories by default.
It is more likely that the software’s main directory ( `C:\Program Files\My Program` in our example) or subdirectory ( `C:\Program Files\My Program\My Service\`) is misconfigured, allowing us to plant a malicious `My.exe` binary.

Although this vulnerability requires a specific combination of requirements, it is easily exploitable and a privilege escalation attack vector worth considering.