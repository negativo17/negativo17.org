Oh no, another Nvidia driver repository? Why? This repository reflects my personal view for the way the driver should be packaged for Fedora and CentOS/RHEL. It's somewhat different from ELRepo repositories for RHEL/CentOS and from RPMFusion packages for Fedora.

# Repository installation

To install the repository on a supported Fedora distribution, run as root the following command:

```
yum config-manager --add-repo=https://negativo17.org/repos/fedora-nvidia.repo
```

To install the repository on CentOS/RHEL:

```
yum-config-manager --add-repo=https://negativo17.org/repos/epel-nvidia.repo
```

Please note that the driver will show up only if your system matches one of the PCI ID supported by the driver. Otherwise, only the other Nvidia programs (mostly for CUDA development) will show up in the software center.

# What's different?

First of all the packaging is a lot simplified; more stuff is compiled from source, smaller packages and more options. This packages try to comply as maximum to the Fedora Packaging Guidelines; which means they have debuginfo packages, default Fedora's GCC compile time options (where possible) and standard locations for binaries, data and docs.

What follows below, is a detailed explanation of all the "differences" from the various Nvidia driver packages that I was able to spot on the web and a detailed description on how to install components, etc.

# Nvidia drivers

## Packaging

`nvidia-settings`, `nvidia-persistenced`, `nvidia-xconfig` and `nvidia-modprobe` are compiled from source.

* All RPM filters except for GL and OpenCL libraries have been removed, so there is no weird dependency option in the SPEC file. RPM pulls in all correct requirements on its own. This is to avoid pulling in the Nvidia drivers instead of the Mesa libraries or in place of the new open source OpenCL support that's in Fedora.
* Simplified packaging with much simpler and readable SPEC file.
* Dependency on `libva-vdpau-driver`. So in Totem, or any other VA-API supported application you can benefit from VDPAU acceleration.
* For the main driver component, sources are generated with a script and inserted individually in the various packages; so it can be easily reproduced just by changing the version and rerunning the script.
* `nvidia-xconfig` is not required on anything that uses the modular X.org directives, as it writes too much in the configuration file (keyboards, monitors, etc.) and the required entries should be written in separate configuration files under `/etc/X11/xorg.conf.d`. The package is still available as it's required to speed up some configuration like multi-monitor setups with SLI Mosaic enabled from the command line, but not installed by default.
* The NVIDIA OpenGL-based Framebuffer Capture (NvFBCOpenGL) libraries (NvFBC and NvIFR) are private APIs that are only available to NVIDIA approved partners for use in remote graphics scenarios (i.e. Steam In-Home Streaming hardware encoding); so they are packaged in another small package called `nvidia-driver-NvFBCOpenGL`.
* The `nvidia-settings` package now builds the external `libXNVCtrl.so` library that can be used to control the graphic cards through the NV-CONTROL extension. This library updates the old and obsolete one in Fedora based on older drivers.
* Starting from version 343.13, the `nvidia-settings` binary is compiled with GTK3 instead of GTK2 on Fedora and RHEL/CentOS 7+.
* The driver can be installed separately from the `nvidia-settings` utility, so if you simply want a working driver and do not care about details, your experience should be as close as possible to the one with open source drivers.

## Versioning

* ELRepo ships 32 bit compatibility libraries in a separate package with x86_64 as the architecture and "32bit" in the name. 32 bit libraries should be like in RPMFusion, with an i686 package installable in parallel with the x86_64 one. There are no other packages in the distribution that are built for x86_64, with "32bit" in their name that contain i686 binaries (!), so Nvidia drivers should not be an exception. So no separate "32bit.x86_64" package for 32 bit libraries also on CentOS/RHEL; just install `nvidia-driver-libs.i686`.
* Versions are not hidden; all packages have the same driver version.
* No alternatives system, only the latest version which integrates CUDA support is available. For older cards `nouveau` works great; and anything below a GeForce 8xxx it's in my opinion too low end to play anything modern. And Quake 3 and Doom 3 work greatly with `nouveau`, so that's not a case!
* The CentOS/RHEL repository contains the "Long Lived Branch version" where less changes occur; while Fedora repositories contains the "Short Lived Branch version". Beta CentOS/RHEL and Fedora's rawhide repositories will contain the "Beta Branch version"

## CUDA support

* CUDA libraries/tools for the driver are split into subpackages. There's no need to install all the CUDA libraries and tools on a system that has only one adapter and is used for occasional gaming or for simple office use. **This can save ~120 MB worth of installed libraries.**
* `nvidia-persistenced` falls in this category as it's not needed on a normal laptop or gaming system.
* Complete packaged CUDA stack has been added for all supported distributions, all the packages provide/require/obsolete the relevant packages in the [Nvidia CUDA repository](http://developer.download.nvidia.com/compute/cuda/repos/); so you can enable this repository along with the official Nvidia CUDA one (x86_64 systems only).

## Kernel modules

* Multiple choice of kernel module packages; `akmod` (RPMFusion) for Fedora and binary `kmod` (Kernel ABI whitelists) for CentOS/RHEL. In addition to this, on both distributions `dkms` packages are available. This way all cases and personal preferences are covered for both distributions.
* Starting from Nvidia driver version 334.16, the Nvidia DDX driver for X can also rely on the `nvidia-modprobe` command in the system to create devices and set permissions, so the new optional package has been added.
* The `nvidia-uvm` module has a soft dependency on the `nvidia` module, making sure that these modules are not included in the `initrd` (thing that would happen by using systemd's configuration (`module-s-load.d`). `udev` rules make sure the module has proper permissions.
* On Fedora, the kernel modules are compressed with XZ, like all the other kernel modules.

## Default configuration

* Dracut options are depending on the distribution; so no more "vga=normal is an obsolete option" at boot. Each distribution gets its own specific GRUB options for booting.
* 96 DPI is written in the default `xorg.conf` config file. Why? Gnome 3 by defaults hard-codes a 96x96 DPI resolution, most of the free drivers do (`intel`, `nouveau`, etc.) as the EDID is almost never reliable (please see the excellent [Adam's Jackson post](https://lists.fedoraproject.org/pipermail/devel/2011-October/157671.html) where he explains this. As an example, if you install the Nvidia drivers on a RHEL/CentOS 6 laptop where you used to have `nouveau` installed (96 DPI hardcoded), the fonts gets 90% of the time supersize and ugly as Gnome 2 and the Nvidia driver do not hard-code 96 DPI like Gnome 3.
* Make X.org NVIDIA Files section to be loaded latest in case there are other packages providing a custom Files section.
* Use new `OutputClass` directive on X.org server 1.16 (and later) to load the driver and do not rely on an edited `/etc/X11/xorg.conf` file. This also removes editing of the `xorg.conf` file from the package scriptlets. This does not hardcode the 96 DPI resolution.
* Add the `IgnoreABI` directive by default on Fedora rawhide builds.

## Kernel modesetting and Wayland support

Kernel mode setting on the `nvidia-drm` module has been disabled by default for various reasons. First of all, Wayland support in the drivers require a patched Wayland which has been refused upstream, and then the driver itself does not expose an FB driver for the console, so you won't see any difference in the terminal output, you will still be limited to VGA.

The Wayland libraries are still included in the builds, as all the dependencies are there but they are not used.

## Vulkan support
￼
On supported releases, the Vulkan loader and libraries are installed automatically and you do not need to do anything to enable support in the drivers. CentOS and Red Hat Enterprise Linux 6 does not have Vulkan.

## Distribution and Nvidia driver version support

Here is a rundown of Nvidia supported drivers and options split by distribution. Basically, CentOS/RHEL will always get a Long Lived branch release if possible, Fedora always a Short Lived branch release, and unreleased distributions will always get a Beta driver.

Shortcode
￼
Optimus laptops
The driver should install and operate cleanly whether you are installing it on a system which has one or more discrete Nvidia cards or an Optimus laptop with an Intel and a Nvidia card. Nothing to do to enable or configure Optimus.

This is up to the point that when the drivers are installed, you can even turn off Optimus on or off in your system Bios (if your laptop allows that) and the only difference you should see is that there's an additional VGA card enabled in your system (check with lspci) and that the Nvidia control panel switches between a PRIME Display, like in this picture:

￼
And a normal RandR managed one, like in this one:

￼
Everything else should not be different from your normal experience.

Limitations with the Nvidia driver
The limitations are the same as provided by the Nvidia driver, this means that if you are running it on an Optimus laptop, the Intel card can never power off. Which means higher power consumption, unfortunately. If you have an Optimus laptop and absolutely need the proprietary drivers, my suggestion is still to disable Optimus in the Bios.

Limitations with the OSS stack
On the contrary, if you use the OSS stack (nouveau/intel) the second card can be powered off if there's no application running on it or display directly connected to one of the card's outputs. That's the best reason to use the OSS drivers at all if you you're not doing serious gaming or 3D work:

$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:IGD:+:Pwr:0000:00:02.0
1:DIS: :DynOff:0000:01:00.0
You also got the nifty selection menu about running your game on the discrete card on Gnome, which is really cool:

￼
It will power up the video card just before launching the process. Launching a program through that menu entry is like starting it from the command line with the DRI_PRIME variable declared. For example, the same as above would be:

$ DRI_PRIME=1 quake3 &
$ sudo cat /sys/kernel/debug/vgaswitcheroo/switch
0:IGD:+:Pwr:0000:00:02.0
1:DIS: :DynPwr:0000:01:00.0
As you can see, the discrete video card is turned on. For Steam, you still need to edit each of your game to run on the Nvidia card:

￼
SLI systems
SLI is now enabled by default with the Auto profile, there's nothing to do if you have a SLI system. If you need any different SLI option (AA, SFR, etc.), just override it in X.org configuration files.

Nouveau fallback
With the new expanded OutputClass support for X, as carried out by Hans, it's now super easy to switch to the OSS stack if the proprietary Nvidia driver somehow does not work. No user space component is touched, as soon as the Nvidia kernel module is not loaded (check on /sys/module/nvidia), the desktop starts with the normal OSS components you get with a normal installation. Thanks to all the work done on libglvnd, the libraries loaded are the correct one for the driver you are running.

This means that the performance of the Nvidia card would be abysmal, but still you would get a nice desktop and browser to Google around for answers on how to fix it :).

Sample installation
Here is an example. Let's assume you have a freshly installed CentOS system with a recent Nvidia GPU and you want to:

Install the driver for gaming
Play Vulkan enabled games
Want to be comfortable with the Nvidia control panel
Play 32 bit games (Vulkan included) on a 64 bit system
Play 32 bit Vulkan games on a 64 bit system
# yum install nvidia-driver nvidia-driver-libs.i686
Last metadata expiration check: 0:05:30 ago on Tue 05 Feb 2019 06:52:50 AM CET.
Dependencies resolved.
================================================================================
 Package                   Arch     Version           Repository           Size
================================================================================
Installing:
 nvidia-driver             x86_64   3:415.27-4.fc29   fedora-multimedia   2.4 M
 nvidia-driver-libs        i686     3:415.27-4.fc29   fedora-multimedia    18 M
Installing dependencies:
 akmod-nvidia              x86_64   3:415.27-1.fc29   fedora-multimedia    10 M
 nvidia-driver-cuda-libs   x86_64   3:415.27-4.fc29   fedora-multimedia    25 M
 nvidia-driver-libs        x86_64   3:415.27-4.fc29   fedora-multimedia    34 M
 nvidia-kmod-common        noarch   3:415.27-1.fc29   fedora-multimedia    10 k
 akmods                    noarch   0.5.6-17.fc29     updates              22 k
 egl-wayland               x86_64   1.1.1-3.fc29      updates              29 k
 kernel-devel              x86_64   4.20.6-200.fc29   updates              13 M
 mesa-libEGL               i686     18.2.8-1.fc29     updates             112 k
 mesa-libgbm               i686     18.2.8-1.fc29     updates              39 k
 libglvnd-egl              i686     1:1.1.0-2.fc29    fedora               45 k
 libglvnd-gles             i686     1:1.1.0-2.fc29    fedora               32 k
 libglvnd-opengl           i686     1:1.1.0-2.fc29    fedora               37 k
 libglvnd-opengl           x86_64   1:1.1.0-2.fc29    fedora               39 k
 libva-vdpau-driver        x86_64   0.7.4-22.fc29     fedora               60 k
 libwayland-server         i686     1.16.0-1.fc29     fedora               38 k

Transaction Summary
================================================================================
Install  17 Packages

Total download size: 103 M
Installed size: 385 M
Is this ok [y/N]:
As you can see, this system has akmod enabled kernel modules and libraries for running 32 bit applications. The amount of data to download for the drivers is really small compared to packages that contain CUDA libraries and tools.

Package installation
If you are booting the system in UEFI mode; as a prerequisite to installing any external module (not built into the kernel package), you have to disable UEFI Secure Boot in the system configuration. All modules contained in the kernel package are signed with keys that are generated during build and deleted when packaging. If you want to preserve Secure Boot, you need to sign the modules yourself and import the keys into your hardware module. Doing so is out of scope here; if you need a decent guide just follow Red Hat's guide for signing kernel modules.

First of all remove all the Nvidia drivers you might have on your system due to RPMFusion, ELRepo, or the Nvidia CUDA repository. This is usually accomplished with the following root command:

yum -y remove *nvidia*
Then, to install the Nvidia driver and its control panel utility in CentOS/RHEL with the binary kABI (Kernel ABI whitelist) module (this is the default on CentOS/RHEL), perform the following command:

yum -y install nvidia-driver nvidia-settings
To do the same in Fedora, using akmod modules, perform the following command:

dnf -y install nvidia-driver nvidia-settings
Specific driver installations
For both Fedora and CentOS/RHEL distributions it's possible to install additional packages and / or variant of the basic kernel modules. This paragraph contains some examples. Make sure you have the EPEL repository enabled if you plan to use DKMS modules on CentOS/RHEL.

akmod kernel module variant (Fedora):

dnf -y install nvidia-driver akmod-nvidia
DKMS kernel module variant (Fedora/CentOS/RHEL):

yum/dnf -y install nvidia-driver dkms-nvidia
To add 32 bit libraries on a 64 bit system (for games or applications like Steam):

yum/dnf -y install nvidia-driver-libs.i686
Additional driver configuration to your system
To add additional configuration to your system, just create the /etc/X11/xorg.conf file. For example:

Section "Device"
  Identifier  "Device0"
  Driver      "nvidia"
  Option      "NoLogo" "true"
  Option      "DPI" "96 x 96"
  Option      "SLI" "Auto"
  Option      "nvidiaXineramaInfoOrder" "DFP-0"
  Option      "metamodes" "GPU-a493fbbb-7d76-86a2-8764-d76d487a75a7.DVI-I-1: nvidia-auto-select +0+0, GPU-c02960a4-be28-d5ce-8b02-be04b5e2550b.DVI-I-1: nvidia-auto-select +1680+0"
  Option      "BaseMosaic" "on"
EndSection
In this example we have 2 video cards with one monitor each, so we enabled SLI, Base Mosaic to have multi monitor support on SLI and make a layout with the second GPU monitor on the right of the first one. Also, we fix the DPI to 96x96, which is the hardcoded default in Gnome and in Open Source drivers.

Configuration for CUDA only systems
Your system might only be used for CUDA development and not require the X server to be running the DDX driver at all, so you might want to tweak the configuration a bit to make the system load (for example) the Intel driver as the main display and just the Nvidia driver for GPU workloads. In this case you have two options.

Option with only CUDA components installed
To install just the CUDA components of the driver and not the OpenGL libraries and all files required by the DDX part of the driver, proceed to install as follows:

# yum install nvidia-driver-cuda
Last metadata expiration check: 0:22:51 ago on Tue 05 Feb 2019 06:52:50 AM CET.
Dependencies resolved.
================================================================================
 Package                   Arch     Version           Repository           Size
================================================================================
Installing:
 nvidia-driver-cuda        x86_64   3:415.27-4.fc29   fedora-multimedia   308 k
Installing dependencies:
 akmod-nvidia              x86_64   3:415.27-1.fc29   fedora-multimedia    10 M
 nvidia-driver-NVML        x86_64   3:415.27-4.fc29   fedora-multimedia   457 k
 nvidia-driver-cuda-libs   x86_64   3:415.27-4.fc29   fedora-multimedia    25 M
 nvidia-kmod-common        noarch   3:415.27-1.fc29   fedora-multimedia    10 k
 nvidia-persistenced       x86_64   3:415.27-2.fc29   fedora-multimedia    40 k
 akmods                    noarch   0.5.6-17.fc29     updates              22 k
 kernel-devel              x86_64   4.20.6-200.fc29   updates              13 M

Transaction Summary
================================================================================
Install  8 Packages

Total download size: 49 M
Installed size: 168 M
Is this ok [y/N]:
As you can see, there are no components providing OpenGL or DDX drivers installed on the system. This will not use any X (or Wayland) configuration compared to what has been installed by default on your system.

On top of this, you can still select what kind of kernel modules you want ot have installed (kABI, akmods and dkms).

Option with DDX components installed
The Intel driver should load the modesetting driver, offload the rendering to the Nvidia driver and not use any monitor attached to the X server for the Nvidia driver.

So in this case, I would change /etc/default/grub to remove the nomodeset parameter to make the Intel KMS driver to load properly, regenerate the Grub config file, reboot and use this xorg.conf:

Section "ServerLayout"
  Identifier "layout"
  Screen 0 "intel"
  Inactive "nvidia"
EndSection

Section "Device"
  Identifier "nvidia"
  Driver "nvidia"
  BusID ""
EndSection

Section "Screen"
  Identifier "nvidia"
  Device "nvidia"
  Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
  Identifier "intel"
  Driver "modesetting"
EndSection

Section "Screen"
  Identifier "intel"
  Device "intel"
EndSection
An example of the above can also be read in the official documentation.

The device file /dev/nvidia0 is normally created when loading the Nvidia driver, so if the X driver is not loaded the device file fileis not created. You can use the nvidia-modprobe command that is in the package with the same name. It contains a SUID binary that creates the device files and set the appropriate permissions when automatic device creation is not available. It is called directly by Nvidia libraries:

$ for i in $(rpm -ql nvidia-driver-libs.x86_64); do
  strings $i | grep nvidia-modprobe > /dev/null && echo $i
done
/usr/lib64/libnvidia-cfg.so.1
/usr/lib64/libnvidia-cfg.so.381.22
/usr/lib64/libnvidia-eglcore.so.381.22
/usr/lib64/libnvidia-glcore.so.381.22
/usr/lib64/libnvidia-glsi.so.381.22 /usr/lib64/vdpau/libvdpau_nvidia.so.1 /usr/lib64/vdpau/libvdpau_nvidia.so.381.22
This requires some testing and adjustments with specifics to your setup, but is definitely possible to use the integrated Intel card and or rely on a system without X installed to run the CUDA components.

CUDA
Packaging
Previously in the repository was included the GPU Deployment kit. This was constructed with NVML (NVIDIA Management Library) headers, docs and samples from a separate tarball. The separate tarball was using a different version number than the drivers and was packaged in the nvidia-driver-NVML and nvidia-driver-NVML-devel packages. Installing these, the gpu-deployment-kit dependency provided by the CUDA repositories was preserved. Starting from CUDA version 8, the NVML header is provided by a CUDA subpackage (cuda-nvml-devel) and no longer provided as part of the GPU Deployment kit.
Included is also the Video Codec SDK (Decoder/Encoder) headers, docs and code samples. Again, this uses a different version than the drivers.
All the libraries are split into subpackages, much like in the original Nvidia CUDA repository. This allows you to install and build software relying on specific components without the need to install all the CUDA toolkit just to satisfy a library dependency. With the new packaging organization, the original cuda-devel and cuda-extra-libs will pull in all the specific subpackages giving you the same situation you are accustomed to. Also, for the same reason, static libraries have been included in each respective devel subpackage.
In addition to the libraries bundled in the CUDA toolkit, also the cuDNN library for distributed neural networks is included in the repository. See the table below for details.
Distribution and CUDA version support
Shortcode
￼
CUDA installations
To install just a runtime CUDA support (required for running CUDA enabled programs), without DDX drivers:

yum -y install cuda nvidia-driver-cuda
To just install packages required for enabling CUDA development:

yum -y install cuda-devel
Or if you just want to enable everything:

dnf/yum -y install nvidia-driver nvidia-driver-cuda cuda-devel
A couple of examples. Just the basic tools:

# yum install cuda
Last metadata expiration check: 0:30:55 ago on Tue 05 Feb 2019 06:52:50 AM CET.
Dependencies resolved.
================================================================================
 Package                  Arch    Version              Repository          Size

================================================================================
Installing:
 cuda                     x86_64  1:10.0.130-1.fc29    fedora-multimedia   17 M
Installing dependencies:
 cuda-libs                x86_64  1:10.0.130-1.fc29    fedora-multimedia  8.6 M
 nvidia-driver-cuda-libs  x86_64  3:415.27-4.fc29      fedora-multimedia   25 M

Transaction Summary
================================================================================
Install  3 Packages

Total download size: 51 M
Installed size: 178 M
Is this ok [y/N]:
The basic tools along with all the libraries (note that the NVML headers are included):

# yum install cuda-devel
Last metadata expiration check: 0:35:09 ago on Tue 05 Feb 2019 06:52:50 AM CET.
Dependencies resolved.
================================================================================
 Package                  Arch    Version              Repository          Size
================================================================================
Installing:
 cuda-devel               x86_64  1:10.0.130-1.fc29    fedora-multimedia  1.6 M
Installing dependencies:
 cuda                     x86_64  1:10.0.130-1.fc29    fedora-multimedia   17 M
 cuda-cublas              x86_64  1:10.0.130-1.fc29    fedora-multimedia   31 M
 cuda-cublas-devel        x86_64  1:10.0.130-1.fc29    fedora-multimedia   32 M
 cuda-cudart              x86_64  1:10.0.130-1.fc29    fedora-multimedia  135 k
 cuda-cudart-devel        x86_64  1:10.0.130-1.fc29    fedora-multimedia  533 k
 cuda-cufft               x86_64  1:10.0.130-1.fc29    fedora-multimedia   64 M
 cuda-cufft-devel         x86_64  1:10.0.130-1.fc29    fedora-multimedia  127 M
 cuda-cupti               x86_64  1:10.0.130-1.fc29    fedora-multimedia  1.4 M
 cuda-cupti-devel         x86_64  1:10.0.130-1.fc29    fedora-multimedia  226 k
 cuda-curand              x86_64  1:10.0.130-1.fc29    fedora-multimedia   38 M
 cuda-curand-devel        x86_64  1:10.0.130-1.fc29    fedora-multimedia   61 M
 cuda-cusolver            x86_64  1:10.0.130-1.fc29    fedora-multimedia   40 M
 cuda-cusolver-devel      x86_64  1:10.0.130-1.fc29    fedora-multimedia   15 M
 cuda-cusparse            x86_64  1:10.0.130-1.fc29    fedora-multimedia   27 M
 cuda-cusparse-devel      x86_64  1:10.0.130-1.fc29    fedora-multimedia   28 M
 cuda-libs                x86_64  1:10.0.130-1.fc29    fedora-multimedia  8.6 M
 cuda-npp-devel           x86_64  1:10.0.130-1.fc29    fedora-multimedia   58 M
 cuda-nvgraph             x86_64  1:10.0.130-1.fc29    fedora-multimedia   68 M
 cuda-nvgraph-devel       x86_64  1:10.0.130-1.fc29    fedora-multimedia   13 k
 cuda-nvjpeg              x86_64  1:10.0.130-1.fc29    fedora-multimedia  372 k
 cuda-nvjpeg-devel        x86_64  1:10.0.130-1.fc29    fedora-multimedia   14 k
 cuda-nvml-devel          x86_64  1:10.0.130-1.fc29    fedora-multimedia   53 k
 cuda-nvrtc               x86_64  1:10.0.130-1.fc29    fedora-multimedia  6.3 M
 cuda-nvrtc-devel         x86_64  1:10.0.130-1.fc29    fedora-multimedia   15 k
 cuda-nvtx                x86_64  1:10.0.130-1.fc29    fedora-multimedia   33 k
 cuda-nvtx-devel          x86_64  1:10.0.130-1.fc29    fedora-multimedia   41 k
 nvidia-driver-NVML       x86_64  3:415.27-4.fc29      fedora-multimedia  457 k
 nvidia-driver-cuda-libs  x86_64  3:415.27-4.fc29      fedora-multimedia   25 M

Transaction Summary
================================================================================
Install  29 Packages

Total download size: 650 M
Installed size: 1.6 G
Is this ok [y/N]:
An example where your CUDA application just uses the CUDA Runtime API and not the kernel runtime:

$ sudo dnf install cuda-cudart
Last metadata expiration check: 0:13:10 ago on Sun Oct 23 13:11:01 2016.
Dependencies resolved.
================================================================================
 Package           Arch         Version               Repository           Size
================================================================================
Installing:
 cuda-cudart       x86_64       1:8.0.44-4.fc24       fedora-nvidia       131 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 131 k
Installed size: 536 k
Is this ok [y/N]:
This will avoid you pulling in all the libraries as before just because you need a single library. This is useful for example for programs that leverage just some part of the CUDA toolkit, like the Nvidia Performance Primitives for image and signal processing in FFmpeg, and similar things.

Bugs
Just open an issue to the specific package on GitHub.
