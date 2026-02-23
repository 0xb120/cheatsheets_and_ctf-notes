---
title: "From Folder Deletion to Admin: Lenovo Vantage (CVE‑2025‑13154) – Compass Security Blog"
source: https://blog.compass-security.com/2026/02/from-folder-deletion-to-admin-lenovo-vantage-cve-2025-13154/
author:
  - John Ostrowski
published:
created: 2026-02-22
description:
tags:
  - clippings/articles
  - Windows
  - privesc
---
# From Folder Deletion to Admin: Lenovo Vantage (CVE‑2025‑13154) – Compass Security Blog

> [!summary]+
> > This blog post details a privilege escalation vulnerability (CVE-2025-13154) in Lenovo Vantage, discovered by Manuel Kiesel from Cyllective AG. The vulnerability stems from the application, running as SYSTEM, deleting a user-modifiable directory.
>
> > Initially, the standard `INDEX_ALLOCATION` trick for arbitrary file deletion to SYSTEM privileges was found to be ineffective on current Windows 11 systems. However, the authors adapted an existing ZDI tool by modifying its C++ code to open a handle to the directory directly, leveraging the arbitrary *folder* deletion primitive that was still present.
>
> > A Proof of Concept (PoC) was developed to reliably exploit this, allowing a regular user to gain administrative privileges on Windows 11. Lenovo's PSIRT team handled the disclosure responsibly, and a patch has since been released to mitigate the vulnerability. The post also includes a disclosure timeline.

identified a potential vulnerability in the software’s file cleanup functionality. You can read about this on [cyllective’s blog](https://cyllective.com/blog/posts/lenovo-vantage). It boils down to the fact that the application, which runs as `SYSTEM`, deletes a directory that can be modified by a regular user:

```powershell
private static bool CleanFiles() {
...
    DeleteDirectory(Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.Windows), "Temp"), false);
...
```

In 2021, Abdelhamid Naceri demonstrated a technique showing how the deletion of user writable folder contents by an elevated account can lead to code execution with elevated privileges. We have abused this issue in the past, including in this case where we used the primitive to gain administrative privileges through a Lenovo vulnerability, tracked as CVE-2022-4569 and described in the [Compass’ blog – Lenovo Update Your Privileges](https://blog.compass-security.com/2023/07/lenovo-update-your-privileges/).

After some investigation, it became clear that the `INDEX_ALLOCATION` trick no longer works on a fully up to date Windows 11 system

## Technical Background – Exploit Primitive

As ZDI explained in their [well written blog post](https://www.zerodayinitiative.com/blog/2022/3/16/abusing-arbitrary-file-deletes-to-escalate-privilege-and-other-great-tricks), it was possible to abuse the MSI install rollback feature to turn an arbitrary folder deletion into a privilege escalation. Since this approach only works when a *directory* can be deleted, they expanded the attack surface and showed how the same idea can also be exploited when only arbitrary *file* deletion is available. The key insight was to transform a file deletion (`DeleteFileA`) into a folder deletion by redirecting the call to the directory metadata attribute (`$INDEX_ALLOCATION`).

As you can see here, when we run the following snippet in PowerShell on a fully up to date Windows 11 system such as 24H2, the `INDEX_ALLOCATION` trick no longer appears to work:

```powershell
# From https://x.com/filip_dragovic/status/1881091708039131441

function Invoke-DeleteFileA {
    param (
        [Parameter(Mandatory)]
        [string]$FilePath
    )

    $signature = @"
    [DllImport("kernel32.dll", CharSet = CharSet.Ansi, SetLastError = true)]
    public static extern bool DeleteFileA(string lpFileName);
"@

    $type = Add-Type -MemberDefinition $signature -Name "Win32Api" -Namespace "Kernel32" -PassThru
    $result = $type::DeleteFileA($FilePath)

    if ($result) {
        Write-Output "File deleted successfully"
    } else {
        $errorCode = [Runtime.InteropServices.Marshal]::GetLastWin32Error()
        Write-Output "Failed to delete file. Error code: $errorCode"
    }
}

# Then run the script. The deletion fails.
PS > Invoke-DeleteFileA -FilePath 'C:\test\folder::$INDEX_ALLOCATION'
Failed to delete file. Error code: 2
```

As a result, using `FolderContentsDeleteToFolderDelete.exe` no longer works for arbitrary file deletes, which left us security analysts a bit disappointed

But then shortly after, we realized that we already had an arbitrary folder deletion primitive and did not need to depend on the `INDEX_ALLOCATION` trick in the first place. With that insight, we adapted the ZDI tool and brought the tool back to life.

In `FolderContentsDeleteToFolderDelete.cpp` we remove the reference to the `INDEX_ALLOCATION` on line 27, and on line 118 we open the handle to the directory instead of the file:

```powershell
Line: FolderContentsDeleteToFolderDelete.cpp
27:   symlinkTarget = std::wstring(L"\\??\\") + symlinkTarget;
118:  hf = op.OpenDirectory(exploitFilePath.c_str(), MAXIMUM_ALLOWED, FILE_SHARE_READ | FILE_SHARE_WRITE, CREATE_ALWAYS);
```