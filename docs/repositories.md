# negativo17.org

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Repositories

A brief rundown of the repositories:

### Steam

A repository that packs the latest Steam for Fedora. This are the same packages that are now integrated in RPMFusion except for the packages used to provide a SteamOS like experience on Fedora or additional controller support.

### Flash plugin

A repository that packs the latest Adobe Flash Plugin for Fedora / CentOS / RHEL that tries to be more "compliant". Much like what Redhat used to do for the Flash plugin in RHEL's supplementary channel.

### CDRtools

A repository for `cdrtools` is available for those that don't think that `cdrkit` works enough well. The packages obsoletes the `cdrkit` packages and plugs into as a replacement for other programs that use it. Linux capabilities are used to avoid having the binaries setuid root. The `cdrtools` source package itselft has been consolidated in the Schily Tools source tarball as of now.

### Nvidia driver, CUDA tools and libraries

A repository that packs the latest Nvidia driver and all its ecosystem of tools: NVENC, CUDA, etc.. For Fedora and CentOS/RHEL; quite different from the other repositories I found around.

### Samsung Unified Linux Driver - Printers & Scanners

A repository that contains the Linux driver for Samsung's line of scanners, printers and multifunction devices. It supports local and network connected devices.

### Spotify client

Repackaged Ubuntu binaries for Fedora and recent CentOS distributions.

### RAR

RAR archiver, packaged for Fedora and CentOS/RHEL.

### Multimedia

A repository that packs all the above repositories all in one along other useful multimedia libraries. So if you want BOTH the multimedia packages and the Nvidia drivers & tools, you can just configure this repository on your systems.

Some of the components included inside the repository:

* HandBrake, the really powerful open source multi threaded video encoder.
* MakeMKV ripper (it can rip encrypted Blue Ray and DVDs) and the `libdvdcss` library.
* CUDA enabled builds of Blender and FFMPeg, additional restricted FFMpeg encoders/decoders and multimedia libraries.
* All Gstreamer plugins for the various Gnome releases including VA-API and Nvidia.
* VideoLAN, MPV.
* The entire VA-API stack for Intel Chipsets (drivers, libraries, QuickSync, etc.).
* Nvidia drivers, CUDA & associated libraries.
* Plex Media Player and media server tools like Sonarr, Radarr, Lidarr, Tautulli, SABnzbd and Jackett.
* The Unifi controller with a bundled MongoDB.
* Steam, SteamOS components and Moonlight client.
* Signal for desktops.
* CDRtools / Schily Tools.
* Spotify client & Spotify ripper.
* Flash and OpenH264 plugins for Firefox.

This repository is **NOT** compatible with RPMFusion, UnitedRPMS, nux or whatever. It overwrites packages from the main repsitories and some CentOS/RHEL secondary ones. The CentOS/RHEL repository also contains 32 bit packages for most of the libraries needed as a dependency, contrary to the EPEL repository.

