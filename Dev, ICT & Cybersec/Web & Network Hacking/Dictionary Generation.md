
>[!info]
>**Precision is generally more important than coverage** when considering a dictionary attack, meaning it is more important to create a lean wordlist of relevant passwords than it is to create an enormous, generic wordlist.

## Public Wordlist

Because of this, many wordlists are based on a common theme, such as popular culture references, specific industries, or geographic regions and refined to contain commonly-used passwords.

Kali Linux includes a number of these dictionary files in the `/usr/share/wordlists/` directory and many more are [hosted online](https://github.com/danielmiessler/SecLists).

- [SecLists](https://github.com/danielmiessler/SecLists) ^a28b1d
- [Assetnote Wordlist](https://wordlists.assetnote.io/) ^b03d62
- https://github.com/nice-registry/all-the-package-names (npm public package names)
- https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056

## Custom Wordlist

Better option is [Creating Custom Wordlists](../../Readwise/Articles/blackbird-eu%20-%20Creating%20Custom%20Wordlists%20for%20Bug%20Bounty%20Targets%20A%20Complete%20Guide.md) based on the specific target.

To build an _effective_ custom wordlist, you should gather data from as many diverse sources as possible:
- [Burpsuite](../Tools/Burpsuite.md) history
- [Crawlers](HTTP%20Recon%20and%20Enumeration.md#Endpoint%20Crawling)

### Wordlist generation tools

Web Specific:
- [cewl](../Tools/cewl.md) [^1], [GAP](../Tools/Burpsuite.md#GAP%20✅), [CO2](../Tools/Burpsuite.md#CO2%20⭐)
- [tok](../Tools/tok.md) and [unfurl](../Tools/unfurl.md) (read and parse the input itself - works better with lists of URLs) [^3]
- [html-tool](../Tools/html-tool.md) and [jsluice](../Tools/jsluice.md) (fetches lists of files or URL, then read and parse the contents)
- [htmlq](../Tools/htmlq.md), [jsluice](../Tools/jsluice.md), and getjswords.py [^4] (read and parse the input itself - works better with HTML and JS files)
- [comb](../Tools/comb.md), [anew](../Tools/anew.md) and [wl](../Tools/wl.md) (to combine wordlists)
- [qsreplace](../Tools/qsreplace.md) (to replace field values with placeholders)
- [undust.py](https://github.com/t3l3machus/undust.py), [fuzzuli](../../Clippings/musana%20-%20musanafuzzuli%20fuzzuli%20is%20a%20url%20fuzzing%20tool%20that%20aims%20to%20find%20critical%20backup%20files%20by%20creating%20a%20dynamic%20wordlist%20based%20on%20the%20domain..md) (to generate lists of possible backup files) 

Password specific:
- [crunch](../Tools/crunch.md)
- https://github.com/shmuelamar/cracken
- https://github.com/Mebus/cupp
- https://github.com/sc0tfree/mentalist
- https://github.com/r3nt0n/bopscrk
- [john (dictionary generation)](../Tools/john.md#Dictionary%20generation)

[^3]: [Extracting URL keywords](../../Readwise/Articles/blackbird-eu%20-%20Creating%20Custom%20Wordlists%20for%20Bug%20Bounty%20Targets%20A%20Complete%20Guide.md#Extracting%20URL%20keywords)

[^4]: [Extracting keywords from JavaScript files](../../Readwise/Articles/blackbird-eu%20-%20Creating%20Custom%20Wordlists%20for%20Bug%20Bounty%20Targets%20A%20Complete%20Guide.md#Extracting%20keywords%20from%20JavaScript%20files)

[^1]: [Extracting in-page keywords](../../Readwise/Articles/blackbird-eu%20-%20Creating%20Custom%20Wordlists%20for%20Bug%20Bounty%20Targets%20A%20Complete%20Guide.md#Extracting%20in-page%20keywords)