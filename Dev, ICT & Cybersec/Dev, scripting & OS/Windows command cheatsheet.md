## Basic Windows Commands

### help

Provides help information for Windows commands.

```powershell
HELP [command]

    command - displays help information on that command.
```

### dir

Displays a list of files and subdirectories in a directory.

```powershell
DIR [drive:][path][filename] [/A[[:]attributes]] [/B] [/C] [/D] [/L] [/N]
  [/O[[:]sortorder]] [/P] [/Q] [/R] [/S] [/T[[:]timefield]] [/W] [/X] [/4]
```

- Dir + Invisible files + system files: `dir /A[DHS]`
- recursive: `dir /S`
- show owner: `dir /Q`
- order by: `dir /O:GN`
- List SFN name: `dir /x` or `dir /-n`

>[!warning] Short vs Long File Name (SFN vs LFN)

**SFN** 

> 8 chars filename + .  3 chars extension (eg. `TEST1234.XYZ`)
- Necessary in FAT file-systems
- Uppercase
- Alphanumeric with some special characters
- No spaces
- Only one period

**LFN**

> 255 Characters, backward compatible with SFN

- `Default.aspx` → `DEFAUL~1.ASP`
- `Web.config` → `WEB~1.CON`

In IIS, this can be abused to enumerate files and directories:
- [Steelcon-2023-Beyond_Microsoft_IIS_Short_File_Name_Disclosure.pdf](https://github.com/irsdl/IIS-ShortName-Scanner/blob/master/presentation/Steelcon-2023-Beyond_Microsoft_IIS_Short_File_Name_Disclosure.pdf)
- [IIS-ShortName-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner/tree/master)
- [IIS Tilde Enumeration Scanner](https://portswigger.net/bappstore/523ae48da61745aaa520ef689e75033b)
-[Microsoft IIS tilde character “~” Vulnerability/Feature – Short File/Folder Name Disclosure](chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://soroush.me/downloadable/microsoft_iis_tilde_character_vulnerability_feature.pdf)

### Copy, move, delete

```powershell
COPY [/D] [/V] [/N] [/Y | /-Y] [/Z] [/L] [/A | /B ] source [/A | /B]
     [+ source [/A | /B] [+ ...]] [destination [/A | /B]]

MOVE [/Y | /-Y] [drive:][path]filename1[,...] destination

DEL [/P] [/F] [/S] [/Q] [/A[[:]attributes]] names
ERASE [/P] [/F] [/S] [/Q] [/A[[:]attributes]] names

```

### fc

Compares two files or sets of files and displays the differences between them.

```powershell
FC [/A] [/C] [/L] [/LBn] [/N] [/OFF[LINE]] [/T] [/U] [/W] [/nnnn]
   [drive1:][path1]filename1 [drive2:][path2]filename2
FC /B [drive1:][path1]filename1 [drive2:][path2]filename2

  /A         Displays only first and last lines for each set of differences.
  /B         Performs a binary comparison.
  /C         Disregards the case of letters.
  /L         Compares files as ASCII text.
  /LBn       Sets the maximum consecutive mismatches to the specified
             number of lines.
  /N         Displays the line numbers on an ASCII comparison.
  /OFF[LINE] Do not skip files with offline attribute set.
  /T         Does not expand tabs to spaces.
  /U         Compare files as UNICODE text files.
  /W         Compresses white space (tabs and spaces) for comparison.
  /nnnn      Specifies the number of consecutive lines that must match
             after a mismatch.
  [drive1:][path1]filename1
             Specifies the first file or set of files to compare.
  [drive2:][path2]filename2
             Specifies the second file or set of files to compare.

```

### type

Displays the contents of a text file or files.

```powershell
TYPE [drive:][path]filename
```

### systeminfo

This tool displays operating system configuration information for a local or remote machine, including service pack levels.

```powershell
SYSTEMINFO [/S system [/U username [/P [password]]]] [/FO format] [/NH]
Parameter List:
    /S      system           Specifies the remote system to connect to.
    /U      [domain\\]user    Specifies the user context under which the command should execute.
    /P      [password]       Specifies the password for the given user context. Prompts for input if omitted.
    /FO     format           Specifies the format in which the output is to be displayed.
                             Valid values: "TABLE", "LIST", "CSV".
    /NH                      Specifies that the "Column Header" should not be displayed in the output.
                             Valid only for "TABLE" and "CSV" formats.
    /?                       Displays this help message.

Examples:
    SYSTEMINFO
    SYSTEMINFO /?
    SYSTEMINFO /S system
    SYSTEMINFO /S system /U user
    SYSTEMINFO /S system /U domain\\user /P password /FO TABLE
    SYSTEMINFO /S system /FO LIST
    SYSTEMINFO /S system /FO CSV /NH

```

### tasklist

This tool displays a list of currently running processes on either a local or remote machine.

```powershell
TASKLIST [/S system [/U username [/P [password]]]]
         [/M [module] | /SVC | /V] [/FI filter] [/FO format] [/NH]

```

### findstr

Searches for strings in files.

```powershell
FINDSTR [/B] [/E] [/L] [/R] [/S] [/I] [/X] [/V] [/N] [/M] [/O] [/P] [/F:file]
        [/C:string] [/G:file] [/D:dir list] [/A:color attributes] [/OFF[LINE]]
        strings [[drive:][path]filename[ ...]]
  /B         Matches pattern if at the beginning of a line.
  /E         Matches pattern if at the end of a line.
  /L         Uses search strings literally.
  /R         Uses search strings as regular expressions.
  /S         Searches for matching files in the current directory and all subdirectories.
  /I         Specifies that the search is not to be case-sensitive.
  /X         Prints lines that match exactly.
  /V         Prints only lines that do not contain a match.
  /N         Prints the line number before each line that matches.
  /M         Prints only the filename if a file contains a match.
  /O         Prints character offset before each matching line.
  /P         Skip files with non-printable characters.
  /OFF[LINE] Do not skip files with offline attribute set.
  /A:attr    Specifies color attribute with two hex digits. See "color /?"
  /F:file    Reads file list from the specified file(/ stands for console).
  /C:string  Uses specified string as a literal search string.
  /G:file    Gets search strings from the specified file(/ stands for console).
  /D:dir     Search a semicolon delimited list of directories
  strings    Text to be searched for.
  [drive:][path]filename    Specifies a file or files to search.

```

---

## Default binary path

`C:\Windows\System32`: 32 bit binaries
`C:\Windows\SysWow64`: 32 bit binaries
`C:\Windows\SysNative`: 64 bit binaries

---

## More content and examples
- [Internal Information Gathering (Windows)](../Web%20&%20Network%20Hacking/Internal%20information%20gathering.md#Windows)
- [Powershell for pentesters](Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md)
- [Tunneling (Windows)](../Web%20&%20Network%20Hacking/Pivoting.md#Tunneling%20(Windows))