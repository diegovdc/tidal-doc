---
title: Windows
permalink: wiki/Windows_choco_install/
layout: wiki
---

**March 15 - STATUS**
- **The windows automated installer currently has problems and should NOT be used until this is resolved.**
- Using the Chocolatey `choco` package manager should be avoided until this problem is resolved.
- Use the Manual Installation instructions below.

**April 26 Update**
We are still working to resolve problems with the automated Chocolatey install. We are also seeing an increase in problems related to the Haskell components. Please follow directions carefully.
- Some users have reported success using Chocolatey `choco` commands to install individual packages. Packages that can be done this way include:
    - SuperCollider
    - Sc3plugins
    - ghc (use version 9.4.4, later or earlier won't work)
    - cabal (use version 3.8.1.0)
    
```powershell
choco install ghc --version=9.4.4
choco install cabal --version=3.8.1.0

# validate versions
ghci --version
cabal --version

choco install SuperCollider
choco install Sc3plugins

# Tidal install
cabal update
cabal v1-install tidal
```

- Once these are installed successfully, follow the manual instructions below for SuperDirt and Pulsar.
- If the cabal commands for Tidal produce any errors, you will need to resolve them. A common error is failure to install the network-3.1.2.8 package:

> failed to install network-3.1.2.8  
> package has a ./configure script. If on windows, this requires a unix compatibility
toolchain such as MinGW+MSYS or Cygwin.

Steps to resolve:
- Add these values to your system PATH environment variable.
```powershell
C:\tools\ghc-9.4.4\mingw\bin
C:\tools\msys64\usr\bin
```
- exit and restart powershell
- remove the previous tidal package install attempt by deleting:
```powershell
C:\Users\<yourUser>\AppData\Roaming\ghc\
C:\Users\<yourUser>\AppData\Roaming\cabal\
```
- run the tidal package install commands:
```
cabal update
cabal v1-install tidal
```

If tidal still doesn't install cleanly, see the [Troubleshooting on Windows](https://tidalcycles.org/docs/troubleshoot/troubleshoot_windows/) page.

---

## Automatic installation

**NOTE: see above.** *Using Chocolatey at this time will install a version of Haskell that is not compatible and you will get errors trying to run Tidal commands.*

This method uses the package manager [Chocolatey](https://chocolatey.org/) and will install everything you need, including required dependencies. Please note that this is a significant install process and takes time, but in the end all components will be ready for use. The installer assumes that these aren't installed already. If you do have some components (SuperCollider, SuperDirt, etc) it is recommended to use Manual install steps for the remaining components (see below).

Components installed via Chocolatey package manager:  
- [git](https://git-scm.com/)
- [SuperCollider](http://supercollider.github.io/)
- [sc3-plugins](http://supercollider.github.io/sc3-plugins/)
- [msys2](https://www.msys2.org/) - (only needed for the choco install)
- [Haskell - ghc](https://www.haskell.org/ghcup/) & [cabal](https://www.haskell.org/cabal/)
- [SuperDirt](https://github.com/musikinformatik/SuperDirt) with the [dirt sample library](https://github.com/musikinformatik/Dirt-Samples) and [Vowel quark](https://github.com/supercollider-quarks/Vowel)
- [Pulsar-dev Editor](https://pulsar-edit.dev/) with TidalCycles plugin package

### Installation procedure

Installation has 3 steps. You may get security pop-up windows for you to accept. Windows 7 users: please review the prep steps outlined at the end of this page.

**I - Starting powershell as administrator**
>    -   Windows 10 - Hold down the windows key
>        and press 'x', then choose Windows PowerShell (admin) in
>        the menu that pops up.
>    -   Windows 7 - Click the start button, type `powershell`, then
>        click with the right mouse button and choose **Run as
>       Administrator**.

**II - Installing Chocolatey: the package manager**

> The [Chocolatey](https://chocolatey.org/) package
> manager is required. If you haven't installed it previously, you can
> get it by running this command:
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

**III - Installing TidalCycles**

> Run the following command to install Tidal Cycles using Chocolatey:
```bash
choco install tidalcycles
```
**Note:** The full install will take 30 - 60 minutes. It is best to let it run to the end, but if it exits without completion or if you need to abort - you can try running this command again. Choco will skip over any package dependencies that are already complete.

After the powershell script is finished, you should review the choco install logs for any errors.  
`C:\ProgramData\chocolatey\logs\chocolatey.log`

**IV - Potential problems and fixes**

- *SuperCollider quarks install failed for SuperDirt, Dirt Samples, and/or Vowel*  
These can be installed manually within the SuperCollider IDE. See the command to execute in the Manual installation section below.
- *Tidal package install failed*
    - You can confirm the status of your tidal install with this command: `cabal info tidal`. If you get a message that "There is no package named tidal" then something went wrong and you need to run these commands (follow the steps in the Manual Install section):
```shell
cabal update
cabal v1-install tidal
```

- *Pulsar install failed*  
Download the installer manually from [Pulsar-dev](https://pulsar-edit.dev/). Once installed, follow the step below to install the TidalCycles plugin package.
- *Pulsar install succeeded but didn't install the TidalCycles plugin package*  
This can done manually from within Pulsar. From the top menu, open the Package Manager, select Install, then search for TidalCycles, and select install. This will install the TidalCycle package into Pulsar. For more details, see the Pulsar page in the "Get a Text Editor" section.
- *Haskell (ghc) or cabal install fails.*  
You can try running the `choco install tidalcycles` command again. Or you can try installing individual components with choco:
```powershell
choco install ghc --version=9.4.4
choco install cabal --version=3.8.1.0
```

- For other problems, see the [Troubleshooting on Windows](../troubleshoot/troubleshoot_windows) page.
-----

## Manual installation

This method is recommended for users who already have some of the components installed. Until the Automated method is available again, please ensure that all components below are installed.

### Haskel
- Install ghcup (Haskell package installer)
    - see [Haskell ghcup](https://www.haskell.org/ghcup/)
    - [YouTube - windows ghcup install](https://www.youtube.com/watch?v=bB4fmQiUYPw)
    - Run this command in Windows Powershell (as admin)
```Powershell
Set-ExecutionPolicy Bypass -Scope Process -Force;[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; try { Invoke-Command -ScriptBlock ([ScriptBlock]::Create((Invoke-WebRequest https://www.haskell.org/ghcup/sh/bootstrap-haskell.ps1 -UseBasicParsing))) -ArgumentList $true } catch { Write-Error $_ }
```
- This should install ghci v9.25. But Tidal 1.9.3+ is only compatible with ghci 9.4.2 / 9.4.4.
- Run these commands from powershell (admin) to get the correct ghc and cabal versions:

```Powershell
ghcup install ghc 9.4.4
ghcup install cabal 3.8.1.0
ghcup set ghc 9.4.4
ghcup set cabal 3.8.1.0

-- Validate
ghci --version  -- 9.4.4
cabal --version -- 3.8.1.0
```
- Be sure to complete the validation steps noted.

### SuperCollider
- See [SuperCollider Downloads](https://supercollider.github.io/downloads)
- [SuperCollider Readme](https://github.com/supercollider/supercollider)
    - [Windows Readme](https://github.com/supercollider/supercollider/blob/develop/README_WINDOWS.md)

### SC3 Plugins

[SC3 Plugins](https://supercollider.github.io/sc3-plugins/) is needed if you want to use any of the synthesizers included with Tidal Cycles. Most of the examples in the documentation will still work, but your installation will likely be incomplete without it. You can skip the installation but be sure to come back later to install it if you want.

### SuperDirt

SuperDirt is the audio engine used by Tidal. Without it, Tidal Cycles will not emit any sound and will not be able to communicate through OSC or MIDI with other synthesizers. To install it, open SuperCollider and run the following command in the interactive editor (press Ctrl+Return to evaluate):

```c
Quarks.checkForUpdates({Quarks.install("SuperDirt", "v1.7.3"); thisProcess.recompile()})
```

The installation will take a little while. You will know when the installation process is done by looking at the logs window. It will likely print something like the following:

```c
Installing SuperDirt
Installing Vowel
Vowel installed
Installing Dirt-Samples
Dirt-Samples installed
SuperDirt installed
compiling class library...
...
(then some blah blah, and finally, something like:)
...

<!--T:28-->
*** Welcome to SuperCollider 3.12.1. *** For help press Ctrl-D.
```

### Tidal Cycles

- Make sure your Haskell environment is correct (above) and that you have ghci v9.4.4. (v9.2.5 and 9.6.1 won't work with Tidal on Windows.)  
- Open `PowerShell` in **administrator mode** (see above).
- Enter the following commands:

```shell
cabal update
cabal v1-install tidal
```
Make sure to use `v1-install`, as `v2-install tidal` *may not work*.
The last command might take some time to complete. Be patient :smile:.

### Pulsar
- See [Pulsar-edit Downloads](https://pulsar-edit.dev/download.html) to download and install.
- OR go to the Pulsar page under Installation > Get a Text Editor  section in the left navigation pane.
- Once you have Pulsar, you need the TidalCycles plugin. Use the Pulsar Package Manager. See details on our Pulsar page.

-----

## Getting Help

If you are having trouble with installation, here are options:
- Review this page carefully and make sure you are following all instructions.  
- For individual component problems - such as SuperCollider and SuperDirt - check their ReadMe pages in GitHub:  
    - [SuperCollider Readme](https://github.com/supercollider/supercollider)
    - [SuperDirt Readme](https://github.com/musikinformatik/SuperDirt)
- [TidalCycles Discord - Installation Help Channel](https://discord.com/channels/779427371270275082/779487905822801930)
    - Try searching this channel to see if your problem has been experienced by others
    - Be sure to post details - what exact problem, error messages, what Windows version, etc.
    - See the "how to ask" channel for more about getting help from our community
- [Forums - Tidal Club](https://club.tidalcycles.org/)  A lot of smart people hang out here.
- Don't get discouraged! Tidal has a complex stack, but these components are all proven, robust and stable. Once it is all working, it rarely needs to have any attention.
----

## Note for Windows 7 users

If you are using Windows 7, some extra preparation is required before installing everything else:

1.  Make sure Windows 7 is up to date with the latest patches.
2.  Install/upgrade to .NET 4.5. You can [download it here](https://www.microsoft.com/en-gb/download/details.aspx?id=30653).
3.  Install Visual C++ redistributable. You can [download it here](https://support.microsoft.com/en-gb/help/2977003/the-latest-supported-visual-c-downloads)

-----

## I've installed Tidal Cycles. What's next?

Look at the sidebar. You will see a list of text editors you can install to interact with Tidal and start playing :smile:.
