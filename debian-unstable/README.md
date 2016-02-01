![Example of Ice](https://raw.githubusercontent.com/scottrice/Ice/master/ice-example.png "Example")

#ice-steamos

ice-steamos is a fork of the original upstream "[Ice](https://github.com/scottrice/Ice)" program by Scott Rice.

Use Steam to play all your old games! Just because old consoles aren't sold anymore doesn't mean you can't enjoy their games. Ice leverages Steam's Big Picture mode to turn it into an emulator frontend (similar to Hyperspin). It accomplishes this by creating folders in specified locations on the user's hard drive, then adding any ROMs that are placed in these folders to Steam as non-Steam games.

The goal of this fork is to fix up Ice and tailor it for use on SteamOS. Debian packaging is in use to automate installation over using pip/python commands. pysteam, psutil, pysteam, and pastebin python packages were converted into Debian packages, which requires adding the SteamOS-Tools repository, noted below. `ice-steamos` follows the latest "stable" release (either upstream or deemed stable by us), and `ice-steamos-unstable`, which follows the latest upstream commits.

#License

The original upstream code, and by extension, ours, is licensed under MIT.

# Getting Started

Ice's upstream official documentation is available at [Getting Started.](http://scottrice.github.io/Ice/getting-started/). 
# Emulators
This fork makes use of as many libretro cores as we can, so that gamepad input (by way of retroarch-joypad-autoconfig) is handled automatically, taking out the need for manual configurations on our part, and the user. However, emulators, such as Dolphin, will need setup as the steam user (or migrated after setup with the desktop user) in Desktop Mode. Making non-libretro cores easier to configure is something that will be looked into. See "depends" lines of  [debian/control](https://github.com/ProfessorKaos64/SteamOS-Tools-Packaging/tree/brewmaster/ice/debian/control) for a list of emulators in use, as well as the [configuration files](https://github.com/ProfessorKaos64/SteamOS-Tools-Packaging/tree/brewmaster/ice) under the SteamOS-Tools build repository.

# Building
Building for specific platforms is located in the [docs/](https://github.com/ProfessorKaos64/Ice/tree/brewmaster/docs) folder. For now, this will just be regarding GNU/Linux platforms.

As this is targeted at SteamOS, build scripts are located in the central [SteamOS-Tools-Packaging repository](https://github.com/ProfessorKaos64/SteamOS-Tools-Packaging) under the "Ice" folder.

# Requirements
This repository **requires** that you have _either_

* [added SteamOS-Tools repositories automatically](https://github.com/ProfessorKaos64/SteamOS-Tools)
* [added SteamOS-Tools repositories manually](https://github.com/ProfessorKaos64/SteamOS-Tools/wiki/SteamOS-Tools-Repository)

# Installation
Please see the releases page for the latest information. You will need an SSH client installed on your remote computer/mobile to connect to the SteamOS machine. 

Stable build:
```
sudo apt-get install ice-steamos openssh-server
```

Unstable build:
```
sudo apt-get install ice-unstable openssh-server
```

There is also a pacakge called [ice-steamos-unstable](http://packages.libregeek.org/SteamOS-Tools/pool/games/i/ice-steamos-unstable/). This package follows upstream closely, with slight modication to logs and data paths locations. This is available via the libregeek repository. The package _will break_ / _replace_ your ice-steamos installation.

# Running Ice
Once Ice is installed, simply run `ice-steamos` as the "steam" user. Because Steam _cannot_ be running at the time you run Ice, you must do the following over SSH (openssh-server can be installed via apt-get). Below is how you need to do this. For instructions on settings the desktop and steam user passwords, please see [this](https://github.com/ValveSoftware/SteamOS/wiki/Setting-steam-and-desktop-user-passwords) wiki page. Substitute "steamos" below for the ip address of your SteamOS machine / computer if you do not have DNS resolution setup on your router or locally.


```
ssh desktop@steamos
sudo systemctl stop lightdm
su - steam
ice-steamos
exit
sudo systemctl start lightdm
```

When you stop lightdm, you will see your SteamOS machine / computer go black. When you start it, Steam BPM will restart once again. Alternatives are being sought, but no other easy way has yet been determined to launch ice, as Steam cannot be running (which is almost always on a SteamOS installation).

# Adding ROMs
When running Ice for the first time the directory `/home/steam/ROMs` will be created containing directories for all supported emulators.

The easiest way to add ROMs to this directory is with [winscp](https://winscp.net/eng/index.php) or rsync, depending on your current platform. These pieces of software allow you to transfer files over ssh. To be able to use ssh as the steam user, you will need to set a password this can be done from the commandline as the desktop user with:

    sudo passwd steam

Once you've done that you can connect to it with winscp with steam as username and with your new password.

If you're on Mac or Linux you can transfer and the content of your local roms directory like this:

    rsync -ave ssh romsdir/NES/ steam@steamos:ROMs/NES/

Or you could use scp for single files:

    scp rom.z64 steam@steamos:ROMs/N64/

In both cases replace steamos with your Steam Machine's IP address and the rom/romdir with what you want to transfer. scp can also handle folders with it's "-r" option in the same way the "cp" command does.

Once you've done this you can run Ice again to add the ROMs to Steam.

# Removing ROMs / Categories added to to SteamOS
Simply remove / relocate the ROM/folder and run Ice again. If there are no games in a given folder, the category will be removed.

# Demo videos
* [Install](https://youtu.be/xqP09mBIwV0)
* [Just gameplay](https://m.youtube.com/watch?v=WfB8F1mrCns_)

# Removal
`ice-steamos` and `ice-unstable` can be uninstalled via apt-get:

```
sudo apt-get remove ice-steamos
```

If you wish to purge the configuration files:

```
sudo apt-get purge ice-steamos
```

# Troubleshooting

## Write access errors
If you receive this message:

```
=========== Starting Ice ===========
Ice cannot run because of issues with your system.

* Ice requires write access too `/home/steam/.local/share/Steam/userdata/21885827/config/grid` to run.

Please resolve these issues and try running Ice again

Close the window, or hit enter to exit...
```

Resolution:

Re-run ice-steamos once more. For some reason, the first time you run ice, this may show up as a false positive. This is being looked into.


## steamwebhelper issues running on a seperate TTY
Error: another process, such as 'steamwebhelper' is still open, preventing Ice from running"

```
`steamwebhelper` cannot be running while Ice is being run. Please resolve these issues and try running Ice again.
Close the window, or hit enter to exit..
```

Resolution:

Switch back to the desktop user by typing "exit," and run 'sudo pkill steamwebhelper'". 'Try running ice-steamos' as the
steam user again. This is the only current reported issue of a process still running after closing lightdm. The bug seems
to present itself only when running ice-steamos on another TTY line, not over SSH.

