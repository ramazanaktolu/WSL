---
title: Configure Linux distributions
description: A reference guide to help you manage and configure multiple Linux distributions running on the Windows Subsystem for Linux.
keywords: BashOnWindows, bash, wsl, windows, windows subsystem for linux, windowssubsystem, ubuntu, wsl.conf, wslconfig
ms.date: 09/27/2021
ms.topic: article
---

# Configure Linux distributions

Windows Subsystem for Linux (WSL) supports running as many different Linux distributions as you would like to install. This can include choosing distributions from the [Microsoft Store](https://aka.ms/wslstore), [importing a custom distribution](./use-custom-distro.md), or [building your own custom distribution](./build-custom-distro.md).

## Ways to run WSL

There are several ways to run a Linux distribution with WSL once it's installed:

1. The number one way that we recommend you run Linux distributions is by [installing Windows Terminal](/windows/terminal/get-started). Using Windows Terminal enables you to open multiple tabs or window panes to display and quickly switch between multiple Linux distributions or other command lines (PowerShell, Command Prompt, PowerShell, Azure CLI, etc). You can fully customize your terminal with unique color schemes, font styles, sizes, background images, and custom keyboard shortcuts. [Learn more.](/windows/terminal)
2. You can directly open your Linux distribution by visiting the Windows Start menu and typing the name of your installed distributions. For example: "Ubuntu". This will open Ubuntu in it's own console window.
3. From Windows Command Prompt or PowerShell, you can enter the name of your installed distribution. For example: `ubuntu`
4. From Windows Command Prompt or PowerShell, you can open your default Linux distribution inside your current command line, by entering: `wsl.exe`.
5. From Windows Command Prompt or PowerShell, you can use your default Linux distribution inside your current command line, without entering a new one, by entering:`wsl [command]`. Replacing `[command]` with a WSL command, such as: `wsl -l -v` to list installed distributions or `wsl pwd` to see where the current directory path is mounted in wsl. From PowerShell, the command `get-date` will provide the date from the Windows file system and `wsl date` will provide the date from the Linux file system.

The method you select should depend on what you're doing. If you've opened a WSL command line within a Windows Prompt or PowerShell window and want to exit, enter the command: `exit`.

![Launch WSL from Start menu](media/start-launch.png)

## List installed distributions

To see a list of the Linux distributions you have installed, enter: `wsl --list` or `wsl -l -v` for a verbose list. To set an installed Linux distribution as the default that is used with the `wsl` command, enter: `wsl -s <DistributionName>` or `wsl --setdefault <DistributionName>`, replacing `<DistributionName>` with the name of the Linux distribution you would like to use. For example, from Powershell, enter: `wsl -s Debian` to set the default distribution to Debian. Now running `wsl npm init` from Powershell will run the `npm init` command in Debian.

## Unregister and reinstall a distribution

While Linux distributions can be installed through the Microsoft Store, they can't be uninstalled through the store.

To unregister and uninstall a WSL distribution:

`wsl --unregister <DistributionName>`  
Unregisters the distribution from WSL so it can be reinstalled or cleaned up. **Caution:** Once unregistered, all data, settings, and software associated with that distribution will be permanently lost.  Reinstalling from the store will install a clean copy of the distribution.

For example:
`wsl --unregister Ubuntu` would remove Ubuntu from the distributions available in WSL.  Running `wsl --list` will reveal that it is no longer listed. To reinstall, find the distribution in the Microsoft Store and select "Launch".

## Run as a specific user

`wsl -u <Username>`, `wsl --user <Username>`

Run WSL as the specified user. Please note that user must exist inside of the WSL distribution.

## Change the default user for a distribution

`<DistributionName> config --default-user <Username>`

Change the default user that for your distribution log-in. The user has to already exist inside the distribution in order to become the default user.

For example:
`ubuntu config --default-user johndoe` would change the default user for the Ubuntu distribution to the "johndoe" user.

> [!NOTE]
> If you are having trouble figuring out the name of your distribution, use the command `wsl -l`.

## Run a specific distribution

`wsl -d <DistributionName>`, `wsl --distribution <DistributionName>`

Run a specified distribution of WSL, can be used to send commands to a specific distribution without having to change your default.

## Configure settings with .wslconfig and wsl.conf

You can configure options for your installed Linux distributions, such as automount options and network configuration, that will automatically be applied every time you launch WSL in two ways:

- Globally for all installed distributions running in WSL 2 mode with a **.wslconfig** file stored in your `%UserProfile%` directory
- On a per-distribution basis with a **wsl.conf** file stored in the `/etc` directory of the distribution

To get to your `%UserProfile%` directory, in PowerShell, use `cd ~` to access your home directory (which is typically your user profile, `C:\Users\<UserName>`) or you can open Windows File Explorer and enter `%UserProfile%` in the address bar. The directory path for globally configuring WSL options will be `C:\Users\<UserName>\.wslconfig`.

To get to the `/etc` directory for an installed distribution, use the distribution's command line with `cd /` to access the root directory, then `ls` to list files or `explorer.exe .` to view in Windows File Explorer. The directory path for configuring WSL options on a per-distribution basis will be `/etc/wsl.conf`.

WSL will detect the existence of these files and read the contents. If the file is missing or malformed (that is, improper markup formatting), WSL will continue to launch as normal.

> [!NOTE]
> Adjusting per-distribution settings with the .wsl.conf file is only available in Windows Build 17093 and later.

## Per distribution configuration options with wsl.conf

The `wsl.conf` sample file below demonstrates some of the configuration options available to add into your distributions:

```console
# Enable extra metadata options by default
[automount]
enabled = true
root = /windir/
options = "metadata,umask=22,fmask=11"
mountFsTab = false

# Enable DNS – even though these are turned on by default, we'll specify here just to be explicit.
[network]
generateHosts = true
generateResolvConf = true
```

When launching multiple Linux shells for the same distribution, you must wait until the Linux subsystem stops running, this can take approximately 8 seconds after closing the last instance of the distribution shell. If you launch a distribution (ie. Ubuntu), modify the wsl.conf file, close the distribution, and then re-launch it. You might assume that your changes to the wsl.conf file have immediately gone into effect. This is not currently the case as the subsystem could still be running. You must wait ~8 seconds for the subsystem to stop before relaunching in order to give enough time for your changes to be picked up. You can check to see whether your Linux distribution (shell) is still running after closing it by using PowerShell with the command: `wsl --list --running`. If no distributions are running, you will receive the response: "There are no running distributions." You can now restart the distribution to see your wsl.conf updates applied.

> [!TIP]
> `wsl --shutdown` is a fast path to restarting WSL 2 distributions, but it will shut down all running distributions, so use wisely.

### Options for wsl.conf

In keeping with .ini conventions, keys are declared under a section.

WSL supports four sections: `automount`, `network`, `interop`, and `user`.

#### automount

Section: `[automount]`

| key | value | default | notes |
|:-----------|:---------|:--------|:------|
| enabled | boolean | true | `true` causes fixed drives (i.e `C:/` or `D:/`) to be automatically mounted with DrvFs under `/mnt`.  `false` means drives won't be mounted automatically, but you could still mount them manually or via `fstab`.                                                                                                             |
| mountFsTab | boolean | true | `true` sets `/etc/fstab` to be processed on WSL start. /etc/fstab is a file where you can declare other filesystems, like an SMB share. Thus, you can mount these filesystems automatically in WSL on start up.                                                                                                                |
| root| String | `/mnt/` | Sets the directory where fixed drives will be automatically mounted. For example, if you have a directory in WSL at `/windir/` and you specify that as the root, you would expect to see your fixed drives mounted at `/windir/c`                                                                                              |
| options | comma-separated list of values | empty string | This value is appended to the default DrvFs mount options string. **Only DrvFs-specific options can be specified.** Options that the mount binary would normally parse into a flag are not supported. If you want to explicitly specify those options, you must include every drive for which you want to do so in /etc/fstab. |

By default, WSL sets the uid and gid to the value of the default user (in Ubuntu distro, the default user is created with uid=1000,gid=1000). If the user specifies a gid or uid option explicitly via this key, the associated value will be overwritten. Otherwise, the default value will always be appended.

**Note:** These options are applied as the mount options for all automatically mounted drives. To change the options for a specific drive only, use /etc/fstab instead.

#### Mount options

Setting different mount options for Windows drives (DrvFs) can control how file permissions are calculated for Windows files. The following options are available:

| Key | Description | Default |
|:----|:----|:----|
|uid| The User ID used for the owner of all files | The default User ID of your WSL distro (On first installation this defaults to 1000)
|gid| The Group ID used for the owner of all files | The default group ID of your WSL distro (On first installation this defaults to 1000)
|umask | An octal mask of permissions to exclude for all files and directories | 000
|fmask | An octal mask of permissions to exclude for all files | 000
|dmask | An octal mask of permissions to exclude for all directories | 000
|metadata | Whether metadata is added to Windows files to support Linux system permissions | disabled
|case | Determines directories treated as case sensitive and whether new directories created with WSL will have the flag set. See [case sensitivity](./case-sensitivity.md) for a detailed explanation of the options. | `off`

> [!NOTE]
> The permission masks are put through a logical OR operation before being applied to files or directories.

#### network

Section label: `[network]`

| key | value | default | notes|
|:----|:----|:----|:----|
| generateHosts | boolean | `true` | `true` sets WSL to generate `/etc/hosts`. The `hosts` file contains a static map of hostnames corresponding IP address. |
| generateResolvConf | boolean | `true` | `true` set WSL to generate `/etc/resolv.conf`. The `resolv.conf` contains a DNS list that are capable of resolving a given hostname to its IP address. | 

#### interop

Section label: `[interop]`

These options are available in Insider Build 17713 and later.

| key | value | default | notes|
|:----|:----|:----|:----|
| enabled | boolean | `true` | Setting this key will determine whether WSL will support launching Windows processes. |
| appendWindowsPath | boolean | `true` | Setting this key will determine whether WSL will add Windows path elements to the $PATH environment variable. |

#### user

Section label: `[user]`

These options are available in Build 18980 and later.

| key | value | default | notes|
|:----|:----|:----|:----|
| default | string | The initial username created on first run | Setting this key specifies which user to run as when first starting a WSL session. |

#### User preview options

These options are only available in the latest preview builds if you are on the latest builds of the [Windows Insiders program](https://insider.windows.com/getting-started).

##### boot

Section label: `[boot]`

| key | value | default | notes|
|:----|:----|:----|:----|
| command | string | "" | A string of the command that you would like to run when the WSL instance starts. This command is run as the root user. e.g: `service docker start` |

## Global configuration options with .wslconfig

You can add a file named `.wslconfig` to your Windows home directory (e.g: `C:\Users\crloewen\.wslconfig`) to control global WSL options across Linux distributions. Please see the sample file below as an example. 

```console
[wsl2]
kernel=C:\\temp\\myCustomKernel
memory=4GB # Limits VM memory in WSL 2 to 4 GB
processors=2 # Makes the WSL 2 VM use two virtual processors
```

> [!NOTE]
> Global configuration options with `.wslconfig` in only available for distributions running as WSL 2 in Windows Build 19041 and later. Keep in mind you may need to run `wsl --shutdown` to shut down the WSL 2 VM and then restart your WSL instance for these changes to take affect.

This file can contain the following options:

### Options for .wslconfig

Section label: `[wsl2]`

These settings affect the VM that powers any WSL 2 distribution.

| key | value | default | notes|
|:----|:----|:----|:----|
| kernel | string | The Microsoft built kernel provided inbox | An absolute Windows path to a custom Linux kernel. |
| memory | size | 50% of total memory on Windows or 8GB, whichever is less; on builds before 20175: 80% of your total memory on Windows | How much memory to assign to the WSL 2 VM. |
| processors | number | The same number of processors on Windows | How many processors to assign to the WSL 2 VM. |
| localhostForwarding | boolean | `true` | Boolean specifying if ports bound to wildcard or localhost in the WSL 2 VM should be connectable from the host via `localhost:port`. |
| kernelCommandLine | string | Blank | Additional kernel command line arguments. |
| swap | size | 25% of memory size on Windows rounded up to the nearest GB | How much swap space to add to the WSL 2 VM, 0 for no swap file. |
| swapFile | string | `%USERPROFILE%\AppData\Local\Temp\swap.vhdx` | An absolute Windows path to the swap virtual hard disk. |

Entries with the `path` value must be Windows paths with escaped backslashes, e.g: `C:\\Temp\\myCustomKernel`

Entries with the `size` value must be a size followed by a unit, for example `8GB` or `512MB`.

### WSL 2 setting preview options

These options are only available in the latest preview builds if you are on the latest builds of the [Windows Insiders program](https://insider.windows.com/getting-started).

| key | value | default | notes|
|:----|:----|:----|:----|
| guiApplications | boolean | `true` | Boolean to turn on or off support for GUI applications ([WSLg](https://github.com/microsoft/wslg)) in WSL. |
| debugConsole | boolean | `false` | Boolean to turn on an output console Window that shows the contents of `dmesg` upon start of a WSL 2 distro instance. |
| nestedVirtualization | boolean | `true` | Boolean to turn on or off nested virtualization for WSL2. |
| vmIdleTimeout | number | `60000` | The number of milliseconds that a VM is idle, before it is shut down. |