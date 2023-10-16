The pacman package manager is one of the major distinguishing features of Arch Linux. It combines a simple binary package format with an easy-to-use build system. The goal of pacman is to make it possible to easily manage packages, whether they are from the official repositories or the user's own builds. [^arch]

[^arch]: [pacman](https://wiki.archlinux.org/title/pacman), wiki.archlinux.org

# Install, uninstall and update packages

```bash
# Install one or more packages, including dependencies
pacman -S package_name, package_name2 ...

# Remove a single package
pacman -R package_name

# Remove a package and all the dependencies
pacman -Rs package_name

# Sync and upgrade every packages
pacman -Syu
```

# Search packages

>[!info]
>Pacman queries the local package database with the `-Q` flag, the sync database with the `-S` flag and the files database with the `-F` flag. See `pacman -Q --help`, `pacman -S --help` and `pacman -F --help` for the respective suboptions of each flag.

```bash
# Search for packages in the database, searching both in packages' names and descriptions:
pacman -Ss string1 string2

# Search for already installed packages:
pacman -Qs string1 string2

# To display extensive information about a given package
pacman -Si package_name

# Use -q to make the output quiter
pacman -Ssq string1 string2
```