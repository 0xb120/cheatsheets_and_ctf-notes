---

database-plugin: basic

---

```yaml:dbfolder
name: CTF list
description: List of every CTF's writeup
columns:
  column1:
    input: text
    key: column1
    accessorKey: column1
    label: Column 1
    position: 0
    skipPersist: false
    isHidden: true
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  __file__:
    key: __file__
    id: __file__
    input: markdown
    label: File
    accessorKey: __file__
    isMetadata: true
    skipPersist: false
    isDragDisabled: false
    csvCandidate: true
    position: 0
    isHidden: false
    sortIndex: 1
    width: 193
    isSorted: true
    isSortedDesc: false
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: true
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  newColumn2:
    input: text
    accessorKey: newColumn2
    key: newColumn2
    id: newColumn2
    label: New Column 2
    position: 100
    skipPersist: false
    isHidden: true
    sortIndex: -1
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Platform:
    input: tags
    accessorKey: Platform
    key: Platform
    id: Platform
    label: Platform
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    width: 180
    options:
      - { label: "HackTheBox", value: "HackTheBox", color: "hsl(113,82%,35%)"}
      - { label: "Cyber Apocalypse 2022", value: "Cyber Apocalypse 2022", color: "hsl(113,82%,35%)"}
      - { label: "m0leCon", value: "m0leCon", color: "hsl(0,53%,51%)"}
      - { label: "PWNX", value: "PWNX", color: "hsl(0,0%,72%)"}
      - { label: "TryHackMe", value: "TryHackMe", color: "hsl(220,73%,53%)"}
      - { label: "X-MAS", value: "X-MAS", color: "hsl(103, 95%, 90%)"}
      - { label: "hxp", value: "hxp", color: "hsl(175, 95%, 90%)"}
      - { label: "justCTF 2023", value: "justCTF 2023", color: "hsl(186, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
      content_alignment: text-align-center
  Difficulty:
    input: select
    accessorKey: Difficulty
    key: Difficulty
    id: Difficulty
    label: Difficulty
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    options:
      - { label: "Easy", value: "Easy", color: "hsl(115,100%,32%)"}
      - { label: "Medium", value: "Medium", color: "hsl(35,100%,41%)"}
      - { label: "Hard", value: "Hard", color: "hsl(0,61%,48%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
      content_alignment: text-align-center
  Category:
    input: select
    accessorKey: Category
    key: Category
    id: Category
    label: Category
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    width: 106
    options:
      - { label: "Misc", value: "Misc", color: "hsl(206,100%,70%)"}
      - { label: "B2R", value: "B2R", color: "hsl(29,92%,66%)"}
      - { label: "Web", value: "Web", color: "hsl(0,100%,73%)"}
      - { label: "Mobile", value: "Mobile", color: "hsl(107,100%,72%)"}
      - { label: "Pwn", value: "Pwn", color: "hsl(276,100%,72%)"}
      - { label: "GamePwn", value: "GamePwn", color: "hsl(56,100%,68%)"}
      - { label: "Blockchain", value: "Blockchain", color: "hsl(30, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
      content_alignment: text-align-center
  Status:
    input: select
    accessorKey: Status
    key: Status
    id: Status
    label: Status
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    width: 141
    options:
      - { label: "3. Complete", value: "3. Complete", color: "hsl(110,100%,72%)"}
      - { label: "1. In progress", value: "1. In progress", color: "hsl(27,100%,71%)"}
      - { label: "2. User", value: "2. User", color: "hsl(228,100%,70%)"}
      - { label: "0. TODO", value: "0. TODO", color: "hsl(267,100%,70%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
      content_alignment: text-align-center
      option_source: manual
  Retired:
    input: checkbox
    accessorKey: Retired
    key: Retired
    id: Retired
    label: Retired
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    width: -93
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
  Tags:
    input: tags
    accessorKey: Tags
    key: Tags
    id: Tags
    label: Tags
    position: 100
    skipPersist: false
    isHidden: false
    sortIndex: -1
    width: 384
    options:
      - { label: "cracking-pdf-files", value: "cracking-pdf-files", color: "hsl(231, 95%, 90%)"}
      - { label: ".mdb,.pst,Windows,anonymous-ftp,credentials-in-wcm,insecure-credentials,runas-privesc", value: ".mdb,.pst,Windows,anonymous-ftp,credentials-in-wcm,insecure-credentials,runas-privesc", color: "hsl(198, 95%, 90%)"}
      - { label: "CSRF,XSS,exploit-chain,flask,pickle-deserialization,zip-slip", value: "CSRF,XSS,exploit-chain,flask,pickle-deserialization,zip-slip", color: "hsl(22, 95%, 90%)"}
      - { label: "ActiveDirectory,Group Policy (GPP/GPO),Kerberoasting,Windows,anonymous-smb", value: "ActiveDirectory,Group Policy (GPP/GPO),Kerberoasting,Windows,anonymous-smb", color: "hsl(15, 95%, 90%)"}
      - { label: "Adminer,CVE-2020-35476,CVE-2021-21311,CVE-2021-25294,CVE-2021-32749,Linux,OpenCats,OpenTSDB,RCE,SSRF,authentication-bypass,credentials-reuse,exploit-chain,fail2ban,hardcoded-credentials,insecure-DB-grants,insecure-credentials,php-deserialization,port-forwarding,whois-spoofing", value: "Adminer,CVE-2020-35476,CVE-2021-21311,CVE-2021-25294,CVE-2021-32749,Linux,OpenCats,OpenTSDB,RCE,SSRF,authentication-bypass,credentials-reuse,exploit-chain,fail2ban,hardcoded-credentials,insecure-DB-grants,insecure-credentials,php-deserialization,port-forwarding,whois-spoofing", color: "hsl(47, 95%, 90%)"}
      - { label: "CVE-2022-22817,RCE,python-PIL", value: "CVE-2022-22817,RCE,python-PIL", color: "hsl(254, 95%, 90%)"}
      - { label: "Android,certificate-pinning-bypass,network_security_config.xml,patching-APK,reversing", value: "Android,certificate-pinning-bypass,network_security_config.xml,patching-APK,reversing", color: "hsl(275, 95%, 90%)"}
      - { label: "Android,hardcoded-credentials,patching-APK,reversing", value: "Android,hardcoded-credentials,patching-APK,reversing", color: "hsl(184, 95%, 90%)"}
      - { label: "MSSQL,Windows,anonymous-smb,credentials-in-history,insecure-credentials,xp_cmdshell", value: "MSSQL,Windows,anonymous-smb,credentials-in-history,insecure-credentials,xp_cmdshell", color: "hsl(262, 95%, 90%)"}
      - { label: "CVE-2009-2265,CVE-2010-2861,ColdFusion,SeImpersonatePrivilege,Windows,arbitrary-file-upload,path-traversal", value: "CVE-2009-2265,CVE-2010-2861,ColdFusion,SeImpersonatePrivilege,Windows,arbitrary-file-upload,path-traversal", color: "hsl(167, 95%, 90%)"}
      - { label: "CVE-2018-7600,Drupal,Drupalgeddon2,Linux,MySQL,credentials-reuse,dirty_sock,hardcoded-credentials,snap-privesc,weak-credentials", value: "CVE-2018-7600,Drupal,Drupalgeddon2,Linux,MySQL,credentials-reuse,dirty_sock,hardcoded-credentials,snap-privesc,weak-credentials", color: "hsl(167, 95%, 90%)"}
      - { label: "esoteric-lang,piet", value: "esoteric-lang,piet", color: "hsl(271, 95%, 90%)"}
      - { label: "authentication-bypass,broken-auth", value: "authentication-bypass,broken-auth", color: "hsl(218, 95%, 90%)"}
      - { label: "debugger-enabled,verbose-log", value: "debugger-enabled,verbose-log", color: "hsl(39, 95%, 90%)"}
      - { label: "RCE,static-eval", value: "RCE,static-eval", color: "hsl(37, 95%, 90%)"}
      - { label: "SSRF,dns-rebinding,evasion", value: "SSRF,dns-rebinding,evasion", color: "hsl(230, 95%, 90%)"}
      - { label: "MySQL,SQL-Injection,addslashes,format-string,real_escape_string,vsprintf", value: "MySQL,SQL-Injection,addslashes,format-string,real_escape_string,vsprintf", color: "hsl(173, 95%, 90%)"}
      - { label: "IDOR,authorization-bypass,broken-auth", value: "IDOR,authorization-bypass,broken-auth", color: "hsl(39, 95%, 90%)"}
      - { label: "JSON2XML,XXE-Injection", value: "JSON2XML,XXE-Injection", color: "hsl(91, 95%, 90%)"}
      - { label: "code-review,command-injection,flask,jail-escape,python-exec", value: "code-review,command-injection,flask,jail-escape,python-exec", color: "hsl(340, 95%, 90%)"}
      - { label: "insecure-credentials,sqlite", value: "insecure-credentials,sqlite", color: "hsl(164, 95%, 90%)"}
      - { label: "Linux,cron,cron-privesc,insecure-file-permissions,phpbash,webshell", value: "Linux,cron,cron-privesc,insecure-file-permissions,phpbash,webshell", color: "hsl(205, 95%, 90%)"}
      - { label: "Drupal,MS15-051,RCE,Windows", value: "Drupal,MS15-051,RCE,Windows", color: "hsl(275, 95%, 90%)"}
      - { label: "pickle-deserialization,python-coding", value: "pickle-deserialization,python-coding", color: "hsl(56, 95%, 90%)"}
      - { label: "PIE,buffer-overflow,code-review,leaking-stack,reversing", value: "PIE,buffer-overflow,code-review,leaking-stack,reversing", color: "hsl(48, 95%, 90%)"}
      - { label: "Elastix,FreePBX,GTFObins,LFI,Linux,RCE,credentials-reuse,nmap-privesc,voip", value: "Elastix,FreePBX,GTFObins,LFI,Linux,RCE,credentials-reuse,nmap-privesc,voip", color: "hsl(126, 95%, 90%)"}
      - { label: "SAM-dump,VHD,Windows,anonymous-smb,credentials-reuse,insecure-credentials,mRemoteNG", value: "SAM-dump,VHD,Windows,anonymous-smb,credentials-reuse,insecure-credentials,mRemoteNG", color: "hsl(288, 95%, 90%)"}
      - { label: "CVE-2021-23639,RCE,md-to-pdf", value: "CVE-2021-23639,RCE,md-to-pdf", color: "hsl(117, 95%, 90%)"}
      - { label: "EternalBlue,MS17-010,Windows,anonymous-smb", value: "EternalBlue,MS17-010,Windows,anonymous-smb", color: "hsl(335, 95%, 90%)"}
      - { label: "Bludit,CVE-2019-14287,CVE-2019-16113,Linux,RCE,brute-force,custom-wordlist,hardcoded-credentials,password-guessing,sudo-privesc", value: "Bludit,CVE-2019-14287,CVE-2019-16113,Linux,RCE,brute-force,custom-wordlist,hardcoded-credentials,password-guessing,sudo-privesc", color: "hsl(349, 95%, 90%)"}
      - { label: "MS10-059,RCE,Windows,insecure-file-upload,web.config", value: "MS10-059,RCE,Windows,insecure-file-upload,web.config", color: "hsl(3, 95%, 90%)"}
      - { label: "CloudMe,Gym-Management-System,RCE,Windows,buffer-overflow,exploit-dev,port-forwarding", value: "CloudMe,Gym-Management-System,RCE,Windows,buffer-overflow,exploit-dev,port-forwarding", color: "hsl(249, 95%, 90%)"}
      - { label: "code-review,deobfuscation,reversing", value: "code-review,deobfuscation,reversing", color: "hsl(142, 95%, 90%)"}
      - { label: ".pcap,IDOR,Linux,cap_setuid-privesc,credentials-reuse", value: ".pcap,IDOR,Linux,cap_setuid-privesc,credentials-reuse", color: "hsl(143, 95%, 90%)"}
      - { label: "Android,Linux,RCE,SSTI,code-review,command-injection,hardcoded-key,insecure-credentials,parameter-injection,reversing", value: "Android,Linux,RCE,SSTI,code-review,command-injection,hardcoded-key,insecure-credentials,parameter-injection,reversing", color: "hsl(180, 95%, 90%)"}
      - { label: "Achat,RCE,Windows,buffer-overflow,credentials-in-registry,credentials-reuse,port-forwarding", value: "Achat,RCE,Windows,buffer-overflow,credentials-in-registry,credentials-reuse,port-forwarding", color: "hsl(57, 95%, 90%)"}
      - { label: "brute-force,code-review,python-coding,reversing", value: "brute-force,code-review,python-coding,reversing", color: "hsl(309, 95%, 90%)"}
      - { label: "crockford", value: "crockford", color: "hsl(88, 95%, 90%)"}
      - { label: "Linux,business-logic-bypass,credentials-reuse,hashcat-custom-rules,insecure-credentials,john-custom-rules", value: "Linux,business-logic-bypass,credentials-reuse,hashcat-custom-rules,insecure-credentials,john-custom-rules", color: "hsl(309, 95%, 90%)"}
      - { label: "Linux,XXE-Injection,bash-history,credentials-reuse,git,git-privesc,pickle-deserialization", value: "Linux,XXE-Injection,bash-history,credentials-reuse,git,git-privesc,pickle-deserialization", color: "hsl(97, 95%, 90%)"}
      - { label: "Windows,anonymous-ftp,arbitrary-file-upload,kernel-exploit,kitrap0d,ms10_015", value: "Windows,anonymous-ftp,arbitrary-file-upload,kernel-exploit,kitrap0d,ms10_015", color: "hsl(6, 95%, 90%)"}
      - { label: "nodejs,race-condition", value: "nodejs,race-condition", color: "hsl(183, 95%, 90%)"}
      - { label: "Unity,memory-editing,memory-inspection,reversing", value: "Unity,memory-editing,memory-inspection,reversing", color: "hsl(232, 95%, 90%)"}
      - { label: "Android,React,hardcoded-key", value: "Android,React,hardcoded-key", color: "hsl(318, 95%, 90%)"}
      - { label: "SSTI,flask,jinja", value: "SSTI,flask,jinja", color: "hsl(177, 95%, 90%)"}
      - { label: "DDNS,Linux,cp-privesc,insecure-credentials,insecure-file-permissions,nsupdate,parameter-injection,php-code-injection", value: "DDNS,Linux,cp-privesc,insecure-credentials,insecure-file-permissions,nsupdate,parameter-injection,php-code-injection", color: "hsl(88, 95%, 90%)"}
      - { label: "OAST,XSS,base_tag-hijacking,cache-poisoning", value: "OAST,XSS,base_tag-hijacking,cache-poisoning", color: "hsl(253, 95%, 90%)"}
      - { label: "cracking-zip-files", value: "cracking-zip-files", color: "hsl(182, 95%, 90%)"}
      - { label: "XSS", value: "XSS", color: "hsl(353, 95%, 90%)"}
      - { label: "CVE-2017-7269,MS14-070,ScStoragePathFromUrl,Windows,kernel-exploit,webdav", value: "CVE-2017-7269,MS14-070,ScStoragePathFromUrl,Windows,kernel-exploit,webdav", color: "hsl(173, 95%, 90%)"}
      - { label: "HTTP-PUT,MS14_058,Windows,arbitrary-file-upload,webdav", value: "HTTP-PUT,MS14_058,Windows,arbitrary-file-upload,webdav", color: "hsl(129, 95%, 90%)"}
      - { label: "LFI,Linux,RCE,SQL-Injection,authentication-bypass,cap_sys_ptrace,control tags,credentials-reuse,gdb-privesc,mPDF,meta-git,process-injection-privesc", value: "LFI,Linux,RCE,SQL-Injection,authentication-bypass,cap_sys_ptrace,control tags,credentials-reuse,gdb-privesc,mPDF,meta-git,process-injection-privesc", color: "hsl(347, 95%, 90%)"}
      - { label: "AST-Injection,__proto__ pollution,__proto__.,nodejs,pug", value: "AST-Injection,__proto__ pollution,__proto__.,nodejs,pug", color: "hsl(124, 95%, 90%)"}
      - { label: "esoteric-lang,malbolge", value: "esoteric-lang,malbolge", color: "hsl(205, 95%, 90%)"}
      - { label: "SQL-Injection,sqlite,webshell", value: "SQL-Injection,sqlite,webshell", color: "hsl(187, 95%, 90%)"}
      - { label: "SeImpersonatePrivilege,Windows,alternate-data-stream,build-RCE,jenkins", value: "SeImpersonatePrivilege,Windows,alternate-data-stream,build-RCE,jenkins", color: "hsl(85, 95%, 90%)"}
      - { label: "buffer-overflow,code-review,reversing", value: "buffer-overflow,code-review,reversing", color: "hsl(166, 95%, 90%)"}
      - { label: "WAR,Windows,default-credentials,tomcat,weak-credentials", value: "WAR,Windows,default-credentials,tomcat,weak-credentials", color: "hsl(262, 95%, 90%)"}
      - { label: "Linux,PHP 8.1.0-dev,backdoor,knife,vi-privesc", value: "Linux,PHP 8.1.0-dev,backdoor,knife,vi-privesc", color: "hsl(329, 95%, 90%)"}
      - { label: "IDOR,XSS,insecure-password-change", value: "IDOR,XSS,insecure-password-change", color: "hsl(49, 95%, 90%)"}
      - { label: "CVE-2007-2447,Linux,RCE,Samba,exploit-dev", value: "CVE-2007-2447,Linux,RCE,Samba,exploit-dev", color: "hsl(35, 95%, 90%)"}
      - { label: "Linux,RCE,SSTI,exploit-dev,flask,python-PIL,python-coding,python-pytesseract,relative-paths-hijacking-privesc", value: "Linux,RCE,SSTI,exploit-dev,flask,python-PIL,python-coding,python-pytesseract,relative-paths-hijacking-privesc", color: "hsl(274, 95%, 90%)"}
      - { label: "CVE-2020-1938,Ghostcat,JSP,JSP-console,RCE-JSP,tomcat", value: "CVE-2020-1938,Ghostcat,JSP,JSP-console,RCE-JSP,tomcat", color: "hsl(322, 95%, 90%)"}
      - { label: "CVE-2008-4250,MS08-067,RCE,Windows", value: "CVE-2008-4250,MS08-067,RCE,Windows", color: "hsl(246, 95%, 90%)"}
      - { label: "RCE,code-review,php-code-injection,php-mail()-RCE,webshell", value: "RCE,code-review,php-code-injection,php-mail()-RCE,webshell", color: "hsl(156, 95%, 90%)"}
      - { label: "RCE,command-injection", value: "RCE,command-injection", color: "hsl(309, 95%, 90%)"}
      - { label: "AlwaysInstallElevated,SSRF,Windows,arbitrary-file-upload,insecure-credentials", value: "AlwaysInstallElevated,SSRF,Windows,arbitrary-file-upload,insecure-credentials", color: "hsl(75, 95%, 90%)"}
      - { label: "PHP-Complex-Syntax,code-review,php-code-injection", value: "PHP-Complex-Syntax,code-review,php-code-injection", color: "hsl(78, 95%, 90%)"}
      - { label: "Linux,RCE,SQL-Injection,authentication-bypass,credentials-reuse,filtering-bypass,insecure-file-upload,port-forwarding,relative-paths-hijacking-privesc,suid", value: "Linux,RCE,SQL-Injection,authentication-bypass,credentials-reuse,filtering-bypass,insecure-file-upload,port-forwarding,relative-paths-hijacking-privesc,suid", color: "hsl(169, 95%, 90%)"}
      - { label: "Android,insecure-password-change", value: "Android,insecure-password-change", color: "hsl(209, 95%, 90%)"}
      - { label: "LFI,Linux,convert-svg,forging-cookies,nodejs,path-traversal", value: "LFI,Linux,convert-svg,forging-cookies,nodejs,path-traversal", color: "hsl(184, 95%, 90%)"}
      - { label: "ERB,SSTI,evasion,regex,ruby", value: "ERB,SSTI,evasion,regex,ruby", color: "hsl(183, 95%, 90%)"}
      - { label: "CKEditor,Linux,MySQL,MySQL-privesc,RCE,UDF-privesc,authentication-bypass,code-review,command-injection,flask,flask-session-cookie-bruteforce,hardcoded-credentials,insecure-credentials,md-to-pdf,python-coding,user-enumeration", value: "CKEditor,Linux,MySQL,MySQL-privesc,RCE,UDF-privesc,authentication-bypass,code-review,command-injection,flask,flask-session-cookie-bruteforce,hardcoded-credentials,insecure-credentials,md-to-pdf,python-coding,user-enumeration", color: "hsl(203, 95%, 90%)"}
      - { label: "Linux,code-review,cracking-shadow-file,exploit-dev,python-coding,reversing", value: "Linux,code-review,cracking-shadow-file,exploit-dev,python-coding,reversing", color: "hsl(21, 95%, 90%)"}
      - { label: "GTFObins,Linux,OpenNetAdmin,code-review,command-injection,credentials-reuse,hardcoded-credentials,nano-privesc,port-forwarding,ssh-keys-cracking", value: "GTFObins,Linux,OpenNetAdmin,code-review,command-injection,credentials-reuse,hardcoded-credentials,nano-privesc,port-forwarding,ssh-keys-cracking", color: "hsl(99, 95%, 90%)"}
      - { label: "Linux,code-review,credentials-reuse,flask,flask-debug_mode,git,git-hook-privesc,hardcoded-credentials,insecure-file-upload,path-traversal,port-forwarding,python-code-injection", value: "Linux,code-review,credentials-reuse,flask,flask-debug_mode,git,git-hook-privesc,hardcoded-credentials,insecure-file-upload,path-traversal,port-forwarding,python-code-injection", color: "hsl(110, 95%, 90%)"}
      - { label: "CVE-2017-1000207,Linux,credentials-reuse,hardcoded-credentials,relative-paths-hijacking-privesc,reversing,snakeyaml,yaml-deserialization", value: "CVE-2017-1000207,Linux,credentials-reuse,hardcoded-credentials,relative-paths-hijacking-privesc,reversing,snakeyaml,yaml-deserialization", color: "hsl(7, 95%, 90%)"}
      - { label: "CVE-2014-6287,CVE-2016-0099,HFS,MS16-032,RCE,Windows", value: "CVE-2014-6287,CVE-2016-0099,HFS,MS16-032,RCE,Windows", color: "hsl(183, 95%, 90%)"}
      - { label: "buffer-overflow", value: "buffer-overflow", color: "hsl(331, 95%, 90%)"}
      - { label: "CVE-2019-17671,CVE-2021-3560,Linux,authorization-bypass,credentials-reuse,polkit-privesc,rocket.chat,wordpress", value: "CVE-2019-17671,CVE-2021-3560,Linux,authorization-bypass,credentials-reuse,polkit-privesc,rocket.chat,wordpress", color: "hsl(282, 95%, 90%)"}
      - { label: "CVE-2017-8291,CVE-2018-16509,GhostButt,Ghostscript,RCE,insecure-file-upload,python-PIL", value: "CVE-2017-8291,CVE-2018-16509,GhostButt,Ghostscript,RCE,insecure-file-upload,python-PIL", color: "hsl(318, 95%, 90%)"}
      - { label: "SQL-Injection,broken-auth,brute-force", value: "SQL-Injection,broken-auth,brute-force", color: "hsl(32, 95%, 90%)"}
      - { label: "Android,certificate-pinning-bypass,frida,reversing", value: "Android,certificate-pinning-bypass,frida,reversing", color: "hsl(59, 95%, 90%)"}
      - { label: "ACL,Linux,RCE,SNMP,SNMP-privesc,SeedDMS,arbitrary-file-upload,cockpit,credentials-reuse,hardcoded-credentials,insecure-directory-permissions,password-guessing,weak-credentials", value: "ACL,Linux,RCE,SNMP,SNMP-privesc,SeedDMS,arbitrary-file-upload,cockpit,credentials-reuse,hardcoded-credentials,insecure-directory-permissions,password-guessing,weak-credentials", color: "hsl(270, 95%, 90%)"}
      - { label: "CVE-2019-12840,Linux,RCE,miniserv,redis,ssh-keys-cracking,webmin", value: "CVE-2019-12840,Linux,RCE,miniserv,redis,ssh-keys-cracking,webmin", color: "hsl(268, 95%, 90%)"}
      - { label: "Group Policy (GPP/GPO),MSSQL,UNC,Windows,anonymous-smb,hardcoded-credentials,insecure-credentials,xp_cmdshell", value: "Group Policy (GPP/GPO),MSSQL,UNC,Windows,anonymous-smb,hardcoded-credentials,insecure-credentials,xp_cmdshell", color: "hsl(277, 95%, 90%)"}
      - { label: "CRFL-injection,CVE-2018-19571,CVE-2018-19585,GitLab,Linux,SSRF,credentials-reuse,docker-breakout,hardcoded-credentials,redis", value: "CRFL-injection,CVE-2018-19571,CVE-2018-19585,GitLab,Linux,SSRF,credentials-reuse,docker-breakout,hardcoded-credentials,redis", color: "hsl(285, 95%, 90%)"}
      - { label: "CVE-2021-33813,Java,Linux,Maven,SSTI,Spring,XXE-Injection,evasion,exploit-chain,jdom2,log-poisoning,path-traversal", value: "CVE-2021-33813,Java,Linux,Maven,SSTI,Spring,XXE-Injection,evasion,exploit-chain,jdom2,log-poisoning,path-traversal", color: "hsl(25, 95%, 90%)"}
      - { label: "Android,Baron Samedit,CVE-2021-3156,Linux,RCE,command-injection,sudo-privesc", value: "Android,Baron Samedit,CVE-2021-3156,Linux,RCE,command-injection,sudo-privesc", color: "hsl(191, 95%, 90%)"}
      - { label: "SQL-Injection", value: "SQL-Injection", color: "hsl(216, 95%, 90%)"}
      - { label: "CVE-2020-14321,CVE-2020-25627,CVE-2020-25629,FreeBSD,GTFObins,Moodle,XSS,credentials-reuse,hardcoded-credentials,moodle-custom-plugin,moodle-log-in-as,pkg-privesc", value: "CVE-2020-14321,CVE-2020-25627,CVE-2020-25629,FreeBSD,GTFObins,Moodle,XSS,credentials-reuse,hardcoded-credentials,moodle-custom-plugin,moodle-log-in-as,pkg-privesc", color: "hsl(140, 95%, 90%)"}
      - { label: "CVE-2020-7384,Linux,command-injection,insecure-file-permissions,msfconsole-privesc", value: "CVE-2020-7384,Linux,command-injection,insecure-file-permissions,msfconsole-privesc", color: "hsl(120, 95%, 90%)"}
      - { label: "CSRF,WSL,Windows,arbitrary-file-upload,credentials-in-history,evasion,insecure-credentials,insecure-password-change", value: "CSRF,WSL,Windows,arbitrary-file-upload,credentials-in-history,evasion,insecure-credentials,insecure-password-change", color: "hsl(105, 95%, 90%)"}
      - { label: "GTFObins,Linux,Shellshock,perl-privesc", value: "GTFObins,Linux,Shellshock,perl-privesc", color: "hsl(226, 95%, 90%)"}
      - { label: "RCE,__init__.py,code-review,flask,flask-debug_mode,path-traversal,python-code-injection,python-tarfile,zip-slip", value: "RCE,__init__.py,code-review,flask,flask-debug_mode,path-traversal,python-code-injection,python-tarfile,zip-slip", color: "hsl(150, 95%, 90%)"}
      - { label: "code-review,reversing", value: "code-review,reversing", color: "hsl(83, 95%, 90%)"}
      - { label: "buffer-overflow,leaking-stack,ret2libc", value: "buffer-overflow,leaking-stack,ret2libc", color: "hsl(88, 95%, 90%)"}
      - { label: "ChromeOS,Linux,autologin,credentials-reuse,hardcoded-credentials,initctl-privesc,insecure-file-permissions,wordpress,wordpress-custom-plugin", value: "ChromeOS,Linux,autologin,credentials-reuse,hardcoded-credentials,initctl-privesc,insecure-file-permissions,wordpress,wordpress-custom-plugin", color: "hsl(319, 95%, 90%)"}
      - { label: "RCE,SQL-Injection,authentication-bypass,code-review,exploit-dev,javascript-code-injection,nodejs", value: "RCE,SQL-Injection,authentication-bypass,code-review,exploit-dev,javascript-code-injection,nodejs", color: "hsl(334, 95%, 90%)"}
      - { label: "LFI,MSSQL,RCE,SQL-Injection,Windows,port-forwarding", value: "LFI,MSSQL,RCE,SQL-Injection,Windows,port-forwarding", color: "hsl(344, 95%, 90%)"}
      - { label: "LDAP-enum,Windows,anonymous-smb,hardcoded-credentials,password-spraying,reversing", value: "LDAP-enum,Windows,anonymous-smb,hardcoded-credentials,password-spraying,reversing", color: "hsl(224, 95%, 90%)"}
      - { label: "LFI,Linux,WAR,cracking-zip-files,lxd-privesc,tomcat", value: "LFI,Linux,WAR,cracking-zip-files,lxd-privesc,tomcat", color: "hsl(35, 95%, 90%)"}
      - { label: "Linux,code-review,credentials-reuse,hardcoded-credentials,php-deserialization,race-condition,temp-file-poisoning", value: "Linux,code-review,credentials-reuse,hardcoded-credentials,php-deserialization,race-condition,temp-file-poisoning", color: "hsl(26, 95%, 90%)"}
      - { label: "crypto,old-ciphers", value: "crypto,old-ciphers", color: "hsl(140, 95%, 90%)"}
      - { label: "CVE-2019-5736,JWT,Linux,RCE,SSRF,authorization-bypass,docker-exec-privesc,insecure-file-permissions,runc-privesc", value: "CVE-2019-5736,JWT,Linux,RCE,SSRF,authorization-bypass,docker-exec-privesc,insecure-file-permissions,runc-privesc", color: "hsl(67, 95%, 90%)"}
      - { label: ".pfx-files,LAPS,Windows,anonymous-smb,cracking-pfx-files,cracking-zip-files,credentials-in-LAPS,credentials-in-history,weak-credentials", value: ".pfx-files,LAPS,Windows,anonymous-smb,cracking-pfx-files,cracking-zip-files,credentials-in-LAPS,credentials-in-history,weak-credentials", color: "hsl(229, 95%, 90%)"}
      - { label: "LFI,code-review,log-poisoning,php-deserialization", value: "LFI,code-review,log-poisoning,php-deserialization", color: "hsl(37, 95%, 90%)"}
      - { label: "Linux,OSINT,insecure-file-permissions,lua,luvit,webshell", value: "Linux,OSINT,insecure-file-permissions,lua,luvit,webshell", color: "hsl(62, 95%, 90%)"}
      - { label: "CVE-2019-16278,GTFObins,Linux,RCE,journalctl-privesc,nostromo,ssh-keys-cracking", value: "CVE-2019-16278,GTFObins,Linux,RCE,journalctl-privesc,nostromo,ssh-keys-cracking", color: "hsl(98, 95%, 90%)"}
      - { label: "LFI,Linux,action.d,dns-enumeration,dns-zone-transfer,evasion,fail2ban,fail2ban-privesc", value: "LFI,Linux,action.d,dns-enumeration,dns-zone-transfer,evasion,fail2ban,fail2ban-privesc", color: "hsl(3, 95%, 90%)"}
      - { label: "CMS-Made-Simple,CVE-2019-9053,Linux,SQL-Injection,relative-paths-hijacking-privesc", value: "CMS-Made-Simple,CVE-2019-9053,Linux,SQL-Injection,relative-paths-hijacking-privesc", color: "hsl(83, 95%, 90%)"}
      - { label: "SQL-Injection,code-review,evasion", value: "SQL-Injection,code-review,evasion", color: "hsl(187, 95%, 90%)"}
      - { label: "OAST,XXE-Injection,evasion,parsers-differential,php-deserialization", value: "OAST,XXE-Injection,evasion,parsers-differential,php-deserialization", color: "hsl(58, 95%, 90%)"}
      - { label: "CVE-2021-3129,Linux,RCE,laravel,laravel-debug_mode,php-deserialization", value: "CVE-2021-3129,Linux,RCE,laravel,laravel-debug_mode,php-deserialization", color: "hsl(6, 95%, 90%)"}
      - { label: ".mdb", value: ".mdb", color: "hsl(183, 95%, 90%)"}
      - { label: ".pst", value: ".pst", color: "hsl(88, 95%, 90%)"}
      - { label: "Windows", value: "Windows", color: "hsl(140, 95%, 90%)"}
      - { label: "anonymous-ftp", value: "anonymous-ftp", color: "hsl(307, 95%, 90%)"}
      - { label: "credentials-in-wcm", value: "credentials-in-wcm", color: "hsl(48, 95%, 90%)"}
      - { label: "insecure-credentials", value: "insecure-credentials", color: "hsl(0, 95%, 90%)"}
      - { label: "runas-privesc", value: "runas-privesc", color: "hsl(9, 95%, 90%)"}
      - { label: "CSRF", value: "CSRF", color: "hsl(280, 95%, 90%)"}
      - { label: "exploit-chain", value: "exploit-chain", color: "hsl(42, 95%, 90%)"}
      - { label: "flask", value: "flask", color: "hsl(347, 95%, 90%)"}
      - { label: "pickle-deserialization", value: "pickle-deserialization", color: "hsl(9, 95%, 90%)"}
      - { label: "zip-slip", value: "zip-slip", color: "hsl(241, 95%, 90%)"}
      - { label: "ActiveDirectory", value: "ActiveDirectory", color: "hsl(235, 95%, 90%)"}
      - { label: "Group Policy (GPP/GPO)", value: "Group Policy (GPP/GPO)", color: "hsl(46, 95%, 90%)"}
      - { label: "Kerberoasting", value: "Kerberoasting", color: "hsl(257, 95%, 90%)"}
      - { label: "anonymous-smb", value: "anonymous-smb", color: "hsl(212, 95%, 90%)"}
      - { label: "Adminer", value: "Adminer", color: "hsl(98, 95%, 90%)"}
      - { label: "CVE-2020-35476", value: "CVE-2020-35476", color: "hsl(326, 95%, 90%)"}
      - { label: "CVE-2021-21311", value: "CVE-2021-21311", color: "hsl(281, 95%, 90%)"}
      - { label: "CVE-2021-25294", value: "CVE-2021-25294", color: "hsl(274, 95%, 90%)"}
      - { label: "CVE-2021-32749", value: "CVE-2021-32749", color: "hsl(10, 95%, 90%)"}
      - { label: "Linux", value: "Linux", color: "hsl(279, 95%, 90%)"}
      - { label: "OpenCats", value: "OpenCats", color: "hsl(255, 95%, 90%)"}
      - { label: "OpenTSDB", value: "OpenTSDB", color: "hsl(92, 95%, 90%)"}
      - { label: "RCE", value: "RCE", color: "hsl(255, 95%, 90%)"}
      - { label: "SSRF", value: "SSRF", color: "hsl(295, 95%, 90%)"}
      - { label: "authentication-bypass", value: "authentication-bypass", color: "hsl(41, 95%, 90%)"}
      - { label: "credentials-reuse", value: "credentials-reuse", color: "hsl(295, 95%, 90%)"}
      - { label: "fail2ban", value: "fail2ban", color: "hsl(39, 95%, 90%)"}
      - { label: "hardcoded-credentials", value: "hardcoded-credentials", color: "hsl(81, 95%, 90%)"}
      - { label: "insecure-DB-grants", value: "insecure-DB-grants", color: "hsl(344, 95%, 90%)"}
      - { label: "php-deserialization", value: "php-deserialization", color: "hsl(326, 95%, 90%)"}
      - { label: "port-forwarding", value: "port-forwarding", color: "hsl(149, 95%, 90%)"}
      - { label: "whois-spoofing", value: "whois-spoofing", color: "hsl(253, 95%, 90%)"}
      - { label: "CVE-2022-22817", value: "CVE-2022-22817", color: "hsl(140, 95%, 90%)"}
      - { label: "python-PIL", value: "python-PIL", color: "hsl(97, 95%, 90%)"}
      - { label: "Android", value: "Android", color: "hsl(19, 95%, 90%)"}
      - { label: "certificate-pinning-bypass", value: "certificate-pinning-bypass", color: "hsl(190, 95%, 90%)"}
      - { label: "network_security_config.xml", value: "network_security_config.xml", color: "hsl(354, 95%, 90%)"}
      - { label: "patching-APK", value: "patching-APK", color: "hsl(140, 95%, 90%)"}
      - { label: "reversing", value: "reversing", color: "hsl(19, 95%, 90%)"}
      - { label: "MSSQL", value: "MSSQL", color: "hsl(90, 95%, 90%)"}
      - { label: "credentials-in-history", value: "credentials-in-history", color: "hsl(178, 95%, 90%)"}
      - { label: "xp_cmdshell", value: "xp_cmdshell", color: "hsl(129, 95%, 90%)"}
      - { label: "CVE-2009-2265", value: "CVE-2009-2265", color: "hsl(92, 95%, 90%)"}
      - { label: "CVE-2010-2861", value: "CVE-2010-2861", color: "hsl(308, 95%, 90%)"}
      - { label: "ColdFusion", value: "ColdFusion", color: "hsl(165, 95%, 90%)"}
      - { label: "SeImpersonatePrivilege", value: "SeImpersonatePrivilege", color: "hsl(293, 95%, 90%)"}
      - { label: "arbitrary-file-upload", value: "arbitrary-file-upload", color: "hsl(168, 95%, 90%)"}
      - { label: "path-traversal", value: "path-traversal", color: "hsl(341, 95%, 90%)"}
      - { label: "CVE-2018-7600", value: "CVE-2018-7600", color: "hsl(304, 95%, 90%)"}
      - { label: "Drupal", value: "Drupal", color: "hsl(83, 95%, 90%)"}
      - { label: "Drupalgeddon2", value: "Drupalgeddon2", color: "hsl(243, 95%, 90%)"}
      - { label: "MySQL", value: "MySQL", color: "hsl(296, 95%, 90%)"}
      - { label: "dirty_sock", value: "dirty_sock", color: "hsl(21, 95%, 90%)"}
      - { label: "snap-privesc", value: "snap-privesc", color: "hsl(142, 95%, 90%)"}
      - { label: "weak-credentials", value: "weak-credentials", color: "hsl(318, 95%, 90%)"}
      - { label: "esoteric-lang", value: "esoteric-lang", color: "hsl(243, 95%, 90%)"}
      - { label: "piet", value: "piet", color: "hsl(166, 95%, 90%)"}
      - { label: "broken-auth", value: "broken-auth", color: "hsl(127, 95%, 90%)"}
      - { label: "debugger-enabled", value: "debugger-enabled", color: "hsl(86, 95%, 90%)"}
      - { label: "verbose-log", value: "verbose-log", color: "hsl(22, 95%, 90%)"}
      - { label: "static-eval", value: "static-eval", color: "hsl(51, 95%, 90%)"}
      - { label: "dns-rebinding", value: "dns-rebinding", color: "hsl(51, 95%, 90%)"}
      - { label: "evasion", value: "evasion", color: "hsl(334, 95%, 90%)"}
      - { label: "addslashes", value: "addslashes", color: "hsl(349, 95%, 90%)"}
      - { label: "format-string", value: "format-string", color: "hsl(204, 95%, 90%)"}
      - { label: "real_escape_string", value: "real_escape_string", color: "hsl(351, 95%, 90%)"}
      - { label: "vsprintf", value: "vsprintf", color: "hsl(277, 95%, 90%)"}
      - { label: "IDOR", value: "IDOR", color: "hsl(44, 95%, 90%)"}
      - { label: "authorization-bypass", value: "authorization-bypass", color: "hsl(117, 95%, 90%)"}
      - { label: "JSON2XML", value: "JSON2XML", color: "hsl(90, 95%, 90%)"}
      - { label: "XXE-Injection", value: "XXE-Injection", color: "hsl(219, 95%, 90%)"}
      - { label: "code-review", value: "code-review", color: "hsl(8, 95%, 90%)"}
      - { label: "command-injection", value: "command-injection", color: "hsl(285, 95%, 90%)"}
      - { label: "jail-escape", value: "jail-escape", color: "hsl(175, 95%, 90%)"}
      - { label: "python-exec", value: "python-exec", color: "hsl(1, 95%, 90%)"}
      - { label: "sqlite", value: "sqlite", color: "hsl(249, 95%, 90%)"}
      - { label: "cron", value: "cron", color: "hsl(1, 95%, 90%)"}
      - { label: "cron-privesc", value: "cron-privesc", color: "hsl(123, 95%, 90%)"}
      - { label: "insecure-file-permissions", value: "insecure-file-permissions", color: "hsl(76, 95%, 90%)"}
      - { label: "phpbash", value: "phpbash", color: "hsl(330, 95%, 90%)"}
      - { label: "webshell", value: "webshell", color: "hsl(321, 95%, 90%)"}
      - { label: "MS15-051", value: "MS15-051", color: "hsl(200, 95%, 90%)"}
      - { label: "python-coding", value: "python-coding", color: "hsl(199, 95%, 90%)"}
      - { label: "PIE", value: "PIE", color: "hsl(163, 95%, 90%)"}
      - { label: "leaking-stack", value: "leaking-stack", color: "hsl(64, 95%, 90%)"}
      - { label: "Elastix", value: "Elastix", color: "hsl(178, 95%, 90%)"}
      - { label: "FreePBX", value: "FreePBX", color: "hsl(116, 95%, 90%)"}
      - { label: "GTFObins", value: "GTFObins", color: "hsl(281, 95%, 90%)"}
      - { label: "LFI", value: "LFI", color: "hsl(101, 95%, 90%)"}
      - { label: "nmap-privesc", value: "nmap-privesc", color: "hsl(7, 95%, 90%)"}
      - { label: "voip", value: "voip", color: "hsl(70, 95%, 90%)"}
      - { label: "SAM-dump", value: "SAM-dump", color: "hsl(212, 95%, 90%)"}
      - { label: "VHD", value: "VHD", color: "hsl(108, 95%, 90%)"}
      - { label: "mRemoteNG", value: "mRemoteNG", color: "hsl(31, 95%, 90%)"}
      - { label: "CVE-2021-23639", value: "CVE-2021-23639", color: "hsl(299, 95%, 90%)"}
      - { label: "md-to-pdf", value: "md-to-pdf", color: "hsl(89, 95%, 90%)"}
      - { label: "EternalBlue", value: "EternalBlue", color: "hsl(35, 95%, 90%)"}
      - { label: "MS17-010", value: "MS17-010", color: "hsl(137, 95%, 90%)"}
      - { label: "Bludit", value: "Bludit", color: "hsl(64, 95%, 90%)"}
      - { label: "CVE-2019-14287", value: "CVE-2019-14287", color: "hsl(216, 95%, 90%)"}
      - { label: "CVE-2019-16113", value: "CVE-2019-16113", color: "hsl(127, 95%, 90%)"}
      - { label: "brute-force", value: "brute-force", color: "hsl(282, 95%, 90%)"}
      - { label: "custom-wordlist", value: "custom-wordlist", color: "hsl(192, 95%, 90%)"}
      - { label: "password-guessing", value: "password-guessing", color: "hsl(56, 95%, 90%)"}
      - { label: "sudo-privesc", value: "sudo-privesc", color: "hsl(271, 95%, 90%)"}
      - { label: "MS10-059", value: "MS10-059", color: "hsl(37, 95%, 90%)"}
      - { label: "insecure-file-upload", value: "insecure-file-upload", color: "hsl(29, 95%, 90%)"}
      - { label: "web.config", value: "web.config", color: "hsl(257, 95%, 90%)"}
      - { label: "CloudMe", value: "CloudMe", color: "hsl(263, 95%, 90%)"}
      - { label: "Gym-Management-System", value: "Gym-Management-System", color: "hsl(79, 95%, 90%)"}
      - { label: "exploit-dev", value: "exploit-dev", color: "hsl(108, 95%, 90%)"}
      - { label: "deobfuscation", value: "deobfuscation", color: "hsl(326, 95%, 90%)"}
      - { label: ".pcap", value: ".pcap", color: "hsl(145, 95%, 90%)"}
      - { label: "cap_setuid-privesc", value: "cap_setuid-privesc", color: "hsl(191, 95%, 90%)"}
      - { label: "SSTI", value: "SSTI", color: "hsl(244, 95%, 90%)"}
      - { label: "hardcoded-key", value: "hardcoded-key", color: "hsl(332, 95%, 90%)"}
      - { label: "parameter-injection", value: "parameter-injection", color: "hsl(118, 95%, 90%)"}
      - { label: "Achat", value: "Achat", color: "hsl(291, 95%, 90%)"}
      - { label: "credentials-in-registry", value: "credentials-in-registry", color: "hsl(269, 95%, 90%)"}
      - { label: "business-logic-bypass", value: "business-logic-bypass", color: "hsl(140, 95%, 90%)"}
      - { label: "hashcat-custom-rules", value: "hashcat-custom-rules", color: "hsl(147, 95%, 90%)"}
      - { label: "john-custom-rules", value: "john-custom-rules", color: "hsl(274, 95%, 90%)"}
      - { label: "bash-history", value: "bash-history", color: "hsl(117, 95%, 90%)"}
      - { label: "git", value: "git", color: "hsl(200, 95%, 90%)"}
      - { label: "git-privesc", value: "git-privesc", color: "hsl(112, 95%, 90%)"}
      - { label: "kernel-exploit", value: "kernel-exploit", color: "hsl(359, 95%, 90%)"}
      - { label: "kitrap0d", value: "kitrap0d", color: "hsl(149, 95%, 90%)"}
      - { label: "ms10_015", value: "ms10_015", color: "hsl(21, 95%, 90%)"}
      - { label: "nodejs", value: "nodejs", color: "hsl(246, 95%, 90%)"}
      - { label: "race-condition", value: "race-condition", color: "hsl(316, 95%, 90%)"}
      - { label: "Unity", value: "Unity", color: "hsl(94, 95%, 90%)"}
      - { label: "memory-editing", value: "memory-editing", color: "hsl(127, 95%, 90%)"}
      - { label: "memory-inspection", value: "memory-inspection", color: "hsl(144, 95%, 90%)"}
      - { label: "React", value: "React", color: "hsl(25, 95%, 90%)"}
      - { label: "jinja", value: "jinja", color: "hsl(283, 95%, 90%)"}
      - { label: "DDNS", value: "DDNS", color: "hsl(176, 95%, 90%)"}
      - { label: "cp-privesc", value: "cp-privesc", color: "hsl(12, 95%, 90%)"}
      - { label: "nsupdate", value: "nsupdate", color: "hsl(56, 95%, 90%)"}
      - { label: "php-code-injection", value: "php-code-injection", color: "hsl(268, 95%, 90%)"}
      - { label: "OAST", value: "OAST", color: "hsl(296, 95%, 90%)"}
      - { label: "base_tag-hijacking", value: "base_tag-hijacking", color: "hsl(168, 95%, 90%)"}
      - { label: "cache-poisoning", value: "cache-poisoning", color: "hsl(330, 95%, 90%)"}
      - { label: "CVE-2017-7269", value: "CVE-2017-7269", color: "hsl(71, 95%, 90%)"}
      - { label: "MS14-070", value: "MS14-070", color: "hsl(150, 95%, 90%)"}
      - { label: "ScStoragePathFromUrl", value: "ScStoragePathFromUrl", color: "hsl(111, 95%, 90%)"}
      - { label: "webdav", value: "webdav", color: "hsl(59, 95%, 90%)"}
      - { label: "HTTP-PUT", value: "HTTP-PUT", color: "hsl(7, 95%, 90%)"}
      - { label: "MS14_058", value: "MS14_058", color: "hsl(321, 95%, 90%)"}
      - { label: "cap_sys_ptrace", value: "cap_sys_ptrace", color: "hsl(320, 95%, 90%)"}
      - { label: "control tags", value: "control tags", color: "hsl(23, 95%, 90%)"}
      - { label: "gdb-privesc", value: "gdb-privesc", color: "hsl(191, 95%, 90%)"}
      - { label: "mPDF", value: "mPDF", color: "hsl(6, 95%, 90%)"}
      - { label: "meta-git", value: "meta-git", color: "hsl(118, 95%, 90%)"}
      - { label: "process-injection-privesc", value: "process-injection-privesc", color: "hsl(105, 95%, 90%)"}
      - { label: "AST-Injection", value: "AST-Injection", color: "hsl(139, 95%, 90%)"}
      - { label: "__proto__ pollution", value: "__proto__ pollution", color: "hsl(173, 95%, 90%)"}
      - { label: "__proto__.", value: "__proto__.", color: "hsl(291, 95%, 90%)"}
      - { label: "pug", value: "pug", color: "hsl(295, 95%, 90%)"}
      - { label: "malbolge", value: "malbolge", color: "hsl(205, 95%, 90%)"}
      - { label: "alternate-data-stream", value: "alternate-data-stream", color: "hsl(170, 95%, 90%)"}
      - { label: "build-RCE", value: "build-RCE", color: "hsl(175, 95%, 90%)"}
      - { label: "jenkins", value: "jenkins", color: "hsl(261, 95%, 90%)"}
      - { label: "WAR", value: "WAR", color: "hsl(0, 95%, 90%)"}
      - { label: "default-credentials", value: "default-credentials", color: "hsl(231, 95%, 90%)"}
      - { label: "tomcat", value: "tomcat", color: "hsl(318, 95%, 90%)"}
      - { label: "PHP 8.1.0-dev", value: "PHP 8.1.0-dev", color: "hsl(106, 95%, 90%)"}
      - { label: "backdoor", value: "backdoor", color: "hsl(12, 95%, 90%)"}
      - { label: "knife", value: "knife", color: "hsl(267, 95%, 90%)"}
      - { label: "vi-privesc", value: "vi-privesc", color: "hsl(352, 95%, 90%)"}
      - { label: "insecure-password-change", value: "insecure-password-change", color: "hsl(47, 95%, 90%)"}
      - { label: "CVE-2007-2447", value: "CVE-2007-2447", color: "hsl(150, 95%, 90%)"}
      - { label: "Samba", value: "Samba", color: "hsl(147, 95%, 90%)"}
      - { label: "python-pytesseract", value: "python-pytesseract", color: "hsl(64, 95%, 90%)"}
      - { label: "relative-paths-hijacking-privesc", value: "relative-paths-hijacking-privesc", color: "hsl(259, 95%, 90%)"}
      - { label: "CVE-2020-1938", value: "CVE-2020-1938", color: "hsl(153, 95%, 90%)"}
      - { label: "Ghostcat", value: "Ghostcat", color: "hsl(208, 95%, 90%)"}
      - { label: "JSP", value: "JSP", color: "hsl(46, 95%, 90%)"}
      - { label: "JSP-console", value: "JSP-console", color: "hsl(228, 95%, 90%)"}
      - { label: "RCE-JSP", value: "RCE-JSP", color: "hsl(22, 95%, 90%)"}
      - { label: "CVE-2008-4250", value: "CVE-2008-4250", color: "hsl(225, 95%, 90%)"}
      - { label: "MS08-067", value: "MS08-067", color: "hsl(227, 95%, 90%)"}
      - { label: "php-mail()-RCE", value: "php-mail()-RCE", color: "hsl(282, 95%, 90%)"}
      - { label: "AlwaysInstallElevated", value: "AlwaysInstallElevated", color: "hsl(157, 95%, 90%)"}
      - { label: "PHP-Complex-Syntax", value: "PHP-Complex-Syntax", color: "hsl(142, 95%, 90%)"}
      - { label: "filtering-bypass", value: "filtering-bypass", color: "hsl(290, 95%, 90%)"}
      - { label: "suid", value: "suid", color: "hsl(141, 95%, 90%)"}
      - { label: "convert-svg", value: "convert-svg", color: "hsl(24, 95%, 90%)"}
      - { label: "forging-cookies", value: "forging-cookies", color: "hsl(298, 95%, 90%)"}
      - { label: "ERB", value: "ERB", color: "hsl(254, 95%, 90%)"}
      - { label: "regex", value: "regex", color: "hsl(272, 95%, 90%)"}
      - { label: "ruby", value: "ruby", color: "hsl(59, 95%, 90%)"}
      - { label: "CKEditor", value: "CKEditor", color: "hsl(288, 95%, 90%)"}
      - { label: "MySQL-privesc", value: "MySQL-privesc", color: "hsl(37, 95%, 90%)"}
      - { label: "UDF-privesc", value: "UDF-privesc", color: "hsl(103, 95%, 90%)"}
      - { label: "flask-session-cookie-bruteforce", value: "flask-session-cookie-bruteforce", color: "hsl(109, 95%, 90%)"}
      - { label: "user-enumeration", value: "user-enumeration", color: "hsl(21, 95%, 90%)"}
      - { label: "cracking-shadow-file", value: "cracking-shadow-file", color: "hsl(37, 95%, 90%)"}
      - { label: "OpenNetAdmin", value: "OpenNetAdmin", color: "hsl(181, 95%, 90%)"}
      - { label: "nano-privesc", value: "nano-privesc", color: "hsl(135, 95%, 90%)"}
      - { label: "ssh-keys-cracking", value: "ssh-keys-cracking", color: "hsl(255, 95%, 90%)"}
      - { label: "flask-debug_mode", value: "flask-debug_mode", color: "hsl(164, 95%, 90%)"}
      - { label: "git-hook-privesc", value: "git-hook-privesc", color: "hsl(2, 95%, 90%)"}
      - { label: "python-code-injection", value: "python-code-injection", color: "hsl(241, 95%, 90%)"}
      - { label: "CVE-2017-1000207", value: "CVE-2017-1000207", color: "hsl(92, 95%, 90%)"}
      - { label: "snakeyaml", value: "snakeyaml", color: "hsl(200, 95%, 90%)"}
      - { label: "yaml-deserialization", value: "yaml-deserialization", color: "hsl(46, 95%, 90%)"}
      - { label: "CVE-2014-6287", value: "CVE-2014-6287", color: "hsl(129, 95%, 90%)"}
      - { label: "CVE-2016-0099", value: "CVE-2016-0099", color: "hsl(110, 95%, 90%)"}
      - { label: "HFS", value: "HFS", color: "hsl(45, 95%, 90%)"}
      - { label: "MS16-032", value: "MS16-032", color: "hsl(178, 95%, 90%)"}
      - { label: "CVE-2019-17671", value: "CVE-2019-17671", color: "hsl(261, 95%, 90%)"}
      - { label: "CVE-2021-3560", value: "CVE-2021-3560", color: "hsl(156, 95%, 90%)"}
      - { label: "polkit-privesc", value: "polkit-privesc", color: "hsl(28, 95%, 90%)"}
      - { label: "rocket.chat", value: "rocket.chat", color: "hsl(310, 95%, 90%)"}
      - { label: "wordpress", value: "wordpress", color: "hsl(88, 95%, 90%)"}
      - { label: "CVE-2017-8291", value: "CVE-2017-8291", color: "hsl(217, 95%, 90%)"}
      - { label: "CVE-2018-16509", value: "CVE-2018-16509", color: "hsl(33, 95%, 90%)"}
      - { label: "GhostButt", value: "GhostButt", color: "hsl(227, 95%, 90%)"}
      - { label: "Ghostscript", value: "Ghostscript", color: "hsl(217, 95%, 90%)"}
      - { label: "frida", value: "frida", color: "hsl(340, 95%, 90%)"}
      - { label: "ACL", value: "ACL", color: "hsl(151, 95%, 90%)"}
      - { label: "SNMP", value: "SNMP", color: "hsl(332, 95%, 90%)"}
      - { label: "SNMP-privesc", value: "SNMP-privesc", color: "hsl(1, 95%, 90%)"}
      - { label: "SeedDMS", value: "SeedDMS", color: "hsl(64, 95%, 90%)"}
      - { label: "cockpit", value: "cockpit", color: "hsl(200, 95%, 90%)"}
      - { label: "insecure-directory-permissions", value: "insecure-directory-permissions", color: "hsl(158, 95%, 90%)"}
      - { label: "CVE-2019-12840", value: "CVE-2019-12840", color: "hsl(176, 95%, 90%)"}
      - { label: "miniserv", value: "miniserv", color: "hsl(119, 95%, 90%)"}
      - { label: "redis", value: "redis", color: "hsl(302, 95%, 90%)"}
      - { label: "webmin", value: "webmin", color: "hsl(241, 95%, 90%)"}
      - { label: "UNC", value: "UNC", color: "hsl(82, 95%, 90%)"}
      - { label: "CRFL-injection", value: "CRFL-injection", color: "hsl(32, 95%, 90%)"}
      - { label: "CVE-2018-19571", value: "CVE-2018-19571", color: "hsl(283, 95%, 90%)"}
      - { label: "CVE-2018-19585", value: "CVE-2018-19585", color: "hsl(126, 95%, 90%)"}
      - { label: "GitLab", value: "GitLab", color: "hsl(285, 95%, 90%)"}
      - { label: "docker-breakout", value: "docker-breakout", color: "hsl(89, 95%, 90%)"}
      - { label: "CVE-2021-33813", value: "CVE-2021-33813", color: "hsl(214, 95%, 90%)"}
      - { label: "Java", value: "Java", color: "hsl(68, 95%, 90%)"}
      - { label: "Maven", value: "Maven", color: "hsl(255, 95%, 90%)"}
      - { label: "Spring", value: "Spring", color: "hsl(0, 95%, 90%)"}
      - { label: "jdom2", value: "jdom2", color: "hsl(0, 95%, 90%)"}
      - { label: "log-poisoning", value: "log-poisoning", color: "hsl(154, 95%, 90%)"}
      - { label: "Baron Samedit", value: "Baron Samedit", color: "hsl(112, 95%, 90%)"}
      - { label: "CVE-2021-3156", value: "CVE-2021-3156", color: "hsl(117, 95%, 90%)"}
      - { label: "CVE-2020-14321", value: "CVE-2020-14321", color: "hsl(78, 95%, 90%)"}
      - { label: "CVE-2020-25627", value: "CVE-2020-25627", color: "hsl(319, 95%, 90%)"}
      - { label: "CVE-2020-25629", value: "CVE-2020-25629", color: "hsl(203, 95%, 90%)"}
      - { label: "FreeBSD", value: "FreeBSD", color: "hsl(146, 95%, 90%)"}
      - { label: "Moodle", value: "Moodle", color: "hsl(154, 95%, 90%)"}
      - { label: "moodle-custom-plugin", value: "moodle-custom-plugin", color: "hsl(220, 95%, 90%)"}
      - { label: "moodle-log-in-as", value: "moodle-log-in-as", color: "hsl(223, 95%, 90%)"}
      - { label: "pkg-privesc", value: "pkg-privesc", color: "hsl(121, 95%, 90%)"}
      - { label: "CVE-2020-7384", value: "CVE-2020-7384", color: "hsl(108, 95%, 90%)"}
      - { label: "msfconsole-privesc", value: "msfconsole-privesc", color: "hsl(278, 95%, 90%)"}
      - { label: "WSL", value: "WSL", color: "hsl(140, 95%, 90%)"}
      - { label: "Shellshock", value: "Shellshock", color: "hsl(282, 95%, 90%)"}
      - { label: "perl-privesc", value: "perl-privesc", color: "hsl(13, 95%, 90%)"}
      - { label: "__init__.py", value: "__init__.py", color: "hsl(152, 95%, 90%)"}
      - { label: "python-tarfile", value: "python-tarfile", color: "hsl(194, 95%, 90%)"}
      - { label: "ret2libc", value: "ret2libc", color: "hsl(38, 95%, 90%)"}
      - { label: "ChromeOS", value: "ChromeOS", color: "hsl(258, 95%, 90%)"}
      - { label: "autologin", value: "autologin", color: "hsl(81, 95%, 90%)"}
      - { label: "initctl-privesc", value: "initctl-privesc", color: "hsl(143, 95%, 90%)"}
      - { label: "wordpress-custom-plugin", value: "wordpress-custom-plugin", color: "hsl(164, 95%, 90%)"}
      - { label: "javascript-code-injection", value: "javascript-code-injection", color: "hsl(246, 95%, 90%)"}
      - { label: "LDAP-enum", value: "LDAP-enum", color: "hsl(356, 95%, 90%)"}
      - { label: "password-spraying", value: "password-spraying", color: "hsl(176, 95%, 90%)"}
      - { label: "lxd-privesc", value: "lxd-privesc", color: "hsl(63, 95%, 90%)"}
      - { label: "temp-file-poisoning", value: "temp-file-poisoning", color: "hsl(275, 95%, 90%)"}
      - { label: "crypto", value: "crypto", color: "hsl(142, 95%, 90%)"}
      - { label: "old-ciphers", value: "old-ciphers", color: "hsl(59, 95%, 90%)"}
      - { label: "CVE-2019-5736", value: "CVE-2019-5736", color: "hsl(96, 95%, 90%)"}
      - { label: "JWT", value: "JWT", color: "hsl(17, 95%, 90%)"}
      - { label: "docker-exec-privesc", value: "docker-exec-privesc", color: "hsl(33, 95%, 90%)"}
      - { label: "runc-privesc", value: "runc-privesc", color: "hsl(29, 95%, 90%)"}
      - { label: ".pfx-files", value: ".pfx-files", color: "hsl(6, 95%, 90%)"}
      - { label: "LAPS", value: "LAPS", color: "hsl(78, 95%, 90%)"}
      - { label: "cracking-pfx-files", value: "cracking-pfx-files", color: "hsl(10, 95%, 90%)"}
      - { label: "credentials-in-LAPS", value: "credentials-in-LAPS", color: "hsl(63, 95%, 90%)"}
      - { label: "OSINT", value: "OSINT", color: "hsl(23, 95%, 90%)"}
      - { label: "lua", value: "lua", color: "hsl(323, 95%, 90%)"}
      - { label: "luvit", value: "luvit", color: "hsl(25, 95%, 90%)"}
      - { label: "CVE-2019-16278", value: "CVE-2019-16278", color: "hsl(17, 95%, 90%)"}
      - { label: "journalctl-privesc", value: "journalctl-privesc", color: "hsl(343, 95%, 90%)"}
      - { label: "nostromo", value: "nostromo", color: "hsl(208, 95%, 90%)"}
      - { label: "action.d", value: "action.d", color: "hsl(107, 95%, 90%)"}
      - { label: "dns-enumeration", value: "dns-enumeration", color: "hsl(185, 95%, 90%)"}
      - { label: "dns-zone-transfer", value: "dns-zone-transfer", color: "hsl(305, 95%, 90%)"}
      - { label: "fail2ban-privesc", value: "fail2ban-privesc", color: "hsl(201, 95%, 90%)"}
      - { label: "CMS-Made-Simple", value: "CMS-Made-Simple", color: "hsl(141, 95%, 90%)"}
      - { label: "CVE-2019-9053", value: "CVE-2019-9053", color: "hsl(1, 95%, 90%)"}
      - { label: "parsers-differential", value: "parsers-differential", color: "hsl(189, 95%, 90%)"}
      - { label: "CVE-2021-3129", value: "CVE-2021-3129", color: "hsl(199, 95%, 90%)"}
      - { label: "laravel", value: "laravel", color: "hsl(22, 95%, 90%)"}
      - { label: "laravel-debug_mode", value: "laravel-debug_mode", color: "hsl(113, 95%, 90%)"}
      - { label: "Blockchain", value: "Blockchain", color: "hsl(215, 95%, 90%)"}
      - { label: "pickle", value: "pickle", color: "hsl(334, 95%, 90%)"}
      - { label: "CTF", value: "CTF", color: "hsl(344, 95%, 90%)"}
      - { label: "Web", value: "Web", color: "hsl(308, 95%, 90%)"}
      - { label: "cracking-kdbx-files", value: "cracking-kdbx-files", color: "hsl(325, 95%, 90%)"}
      - { label: "steganography", value: "steganography", color: "hsl(70, 95%, 90%)"}
      - { label: "blowfish", value: "blowfish", color: "hsl(208, 95%, 90%)"}
      - { label: "image-manipulation", value: "image-manipulation", color: "hsl(350, 95%, 90%)"}
      - { label: "Express", value: "Express", color: "hsl(36, 95%, 90%)"}
      - { label: "-", value: "-", color: "hsl(351, 95%, 90%)"}
      - { label: "ej", value: "ej", color: "hsl(53, 95%, 90%)"}
      - { label: "ejs", value: "ejs", color: "hsl(295, 95%, 90%)"}
      - { label: "CVE-2022-29078", value: "CVE-2022-29078", color: "hsl(32, 95%, 90%)"}
      - { label: "CSP-bypass", value: "CSP-bypass", color: "hsl(37, 95%, 90%)"}
      - { label: "jsfuck", value: "jsfuck", color: "hsl(185, 95%, 90%)"}
      - { label: "rack.session", value: "rack.session", color: "hsl(60, 95%, 90%)"}
      - { label: "sinatra", value: "sinatra", color: "hsl(262, 95%, 90%)"}
      - { label: "server-side_prototype-pollution", value: "server-side_prototype-pollution", color: "hsl(63, 95%, 90%)"}
      - { label: "obfuscation", value: "obfuscation", color: "hsl(272, 95%, 90%)"}
    config:
      enable_media_view: true
      link_alias_enabled: true
      media_width: 100
      media_height: 100
      isInline: false
      task_hide_completed: true
      footer_type: none
      persist_changes: false
config:
  remove_field_when_delete_column: false
  cell_size: compact
  sticky_first_column: true
  group_folder_column: 
  remove_empty_folders: false
  automatically_group_files: false
  hoist_files_with_empty_attributes: true
  show_metadata_created: false
  show_metadata_modified: false
  show_metadata_tasks: false
  show_metadata_inlinks: false
  show_metadata_outlinks: false
  source_data: current_folder
  source_form_result: root
  source_destination_path: /
  frontmatter_quote_wrap: false
  row_templates_folder: /
  current_row_template: zzz_res/templates/HTB-challenge.md
  pagination_size: 25
  enable_js_formulas: false
  formula_folder_path: /
  inline_default: false
  inline_new_position: top
  date_format: yyyy-MM-dd
  datetime_format: "yyyy-MM-dd HH:mm:ss"
  enable_footer: false
  font_size: 16
  metadata_date_format: "yyyy-MM-dd HH:mm:ss"
  implementation: default
  show_metadata_tags: false
filters:
  enabled: false
  conditions:
```