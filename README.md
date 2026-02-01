# Fedora 43 Post Install Guide
Things to do after installing Fedora 42 

## Update and Reboot
* Go into the software center and click on update. Alternatively, you can do:
```
sudo dnf refresh
sudo dnf update
```

## Enabling the RPM Fusion repositories
* Fedora has disabled the repositories for a lot of free and non-free .rpm packages by default. Follow this if you want to use non-free software like Steam, Discord and some multimedia codecs etc. As a general rule of thumb it is advised to do this to get access to many mainstream useful programs.
* If you forgot to enable third party repositories during the initial setup window, enable them by pasting the following into the terminal: 
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`
```
* also while you're at it, install app-stream metadata by
```
sudo dnf group upgrade core
```

## Enable Automatic Updates (Optional)
```
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```
* Customize Auto-Update Behavior (Optional):
```
sudo nano /etc/dnf/automatic.conf
sudo systemctl restart dnf-automatic.timer
```

## Firmware
* If your system supports firmware update delivery through [lvfs](https://fwupd.org/), update your device firmware by:
```
sudo fwupdmgr refresh --force
sudo fwupdmgr get-devices # Lists devices with available updates.
sudo fwupdmgr get-updates # Fetches list of available updates.
sudo fwupdmgr update
```

## [NVIDIA Drivers and Cuda Toolkit](https://developer.nvidia.com/cuda-downloads?target_os=Linux)
* In order for the nvidia driver to work with Secure Boot, you must first install it using Gnome Software.
* You can check how to do it at this [link](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/#gnome-software-integration)
* The kernel headers and development packages for the currently running kernel can be installed with:
```
sudo dnf install kernel-devel-matched kernel-headers
```
* Enable the network repository:
```
sudo dnf config-manager addrepo --from-repofile=https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-$distro.repo`
```
* Clean DNF repository cache:
```
sudo dnf clean expire-cache
```
* Open Kernel Modules
```
sudo dnf install nvidia-open
```
* Proprietary Kernel Modules:
```
sudo dnf install cuda-drivers
```
* Wait for atleast 5 mins before rebooting, to let the kernel module get built.
* Reboot


## [Intel(R) Graphics Compute Runtime for oneAPI Level Zero and OpenCL(TM) Driver](https://github.com/intel/compute-runtime)
```
sudo dnf install intel-compute-runtime intel-level-zero libmfx intel-ocloc intel-opencl libva-utils
```

## Install Intel Thermal Daemon [Thermald](https://github.com/intel/thermal_daemon)
```
sudo dnf install thermald
```

## Media Codecs
* Install these to get proper multimedia playback.
````
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing # Switch to full FFMPEG.
sudo dnf4 group upgrade multimedia
sudo dnf upgrade @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin # Installs gstreamer components. Required if you use Gnome Videos and other dependent applications.
sudo dnf group install -y sound-and-video # Installs useful Sound and Video complement packages.
````

## H/W Video Acceleration
* Helps decrease load on the CPU when watching videos online by alloting the rendering to the dGPU/iGPU. Quite helpful in increasing battery backup on laptops.

### H/W Video Decoding with VA-API 
```
sudo dnf install ffmpeg-libs libva libva-utils
```
* If you have a recent Intel chipset (5th Gen and above) after installing the packages above., Do:
```
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing
sudo dnf install libva-intel-driver
```
* If you have an AMD chipset, after installing the packages above do:
```
sudo dnf swap mesa-va-drivers mesa-va-drivers-freeworld
sudo dnf swap mesa-vdpau-drivers mesa-vdpau-drivers-freeworld
```

## Microsoft Fonts
```
sudo dnf install curl cabextract xorg-x11-font-utils fontconfig 
sudo rpm -i https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm
```

## Visual Studio Code on Linux
* Install the key and yum repository by running the following script:
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\nautorefresh=1\ntype=rpm-md\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null
```
* Then update the package cache and install the package using dnf.
```
sudo dnf check-update
sudo dnf install code
```

## DaVinci Resolve (The program works and detects Intel's integrated graphics)
```
* Installation
sudo dnf install apr apr-util mesa-libGLU libxcrypt-compat fuse fuse-libs libpango-1.0.so.0
unzip DaVinci_Resolve_Studio_20.2.1_Linux.zip
chmod +x ./DaVinci_Resolve_Studio_20.2.1_Linux.run
sudo SKIP_PACKAGE_CHECK=1 ./DaVinci_Resolve_Studio_20.2.1_Linux.run -i

* Repair (/opt/resolve/bin/resolve: symbol lookup error: /usr/lib/libpango-1.0.so.0: undefined symbol: g_once_init_leave_pointer):
cd /opt/resolve/libs
sudo mkdir disabled-libraries
sudo mv libglib* disabled-libraries
sudo mv libgio* disabled-libraries
sudo mv libgmodule* disabled-libraries 
```

## Configuring the system and the GNOME graphical environment

### Set Hostname
* `hostnamectl set-hostname YOUR_HOSTNAME`

### Set UTC Time
* Used to counter time inconsistencies in dual boot systems
```
sudo timedatectl set-local-rtc 1
```

### Default Firefox start page 
* The tweak below will make the start page the default firefox start page instead of [this](https://fedoraproject.org/start)
```
sudo rm -f /usr/lib64/firefox/browser/defaults/preferences/firefox-redhat-default-prefs.js
```

### Enable VAAPI in Firefox "about:config"
```
media.ffmpeg.vaapi.enabled  true
media.navigator.mediadatadecoder_vpx_enabled  true
layers.acceleration.force-enabled true
gfx.webrender.enabled true
gfx.webrender.all true
gfx.x11-egl.force-enabled true
```

### OpenH264 for Firefox
```
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
```
* After this enable the OpenH264 Plugin in Firefox's settings.

### Enable Support for ntfs-3g and exfat:
```
sudo dnf install exfatprogs ntfs-3g
```

### Enable Trim Support
```
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer
```

### Install lm-sensors and detect all sensors
```
sudo dnf install lm_sensors
sudo sensors-detect
```

### Disable `NetworkManager-wait-online.service`
* Disabling it can decrease the boot time by at least ~15s-20s:
```
sudo systemctl disable NetworkManager-wait-online.service
```

### Better Linux Disk Caching & Performance with vm.dirty_ratio & vm.dirty_background_ratio & Sharply reduce swap inclination
```
vm.swappiness=1
vm.vfs_cache_pressure=100
vm.dirty_background_ratio = 5
vm.dirty_background_bytes = 0
vm.dirty_ratio = 10
vm.dirty_bytes = 0
vm.dirty_writeback_centisecs = 500
vm.dirty_expire_centisecs = 3000
```

### Clear RAM memory Cache
```
* To free pagecache:
echo 1 > /proc/sys/vm/drop_caches
* To free dentries and inodes:
echo 2 > /proc/sys/vm/drop_caches
* To free pagecache, dentries and inodes:
echo 3 > /proc/sys/vm/drop_caches
```

### Enable nvidia-modeset 
* Useful if you have a laptop with an Nvidia GPU. Necessary for some PRIME-related interoperability features.
```
sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"
```

### Disable Gnome Software from Startup Apps
* Gnome software autostarts on boot for some reason, even though it is not required on every boot unless you want it to do updates in the background, this takes at least 100MB of RAM upto 900MB (as reported anecdotically). You can stop it from autostarting by:
```
sudo rm /etc/xdg/autostart/org.gnome.Software.desktop
```

### Flatpak
* Fedora doesn't include all non-free flatpaks by default. The command below enables access to all the flathub flatpaks. Particularly useful for users of Fedora KDE and other spins since they do not get the "Enable Third Party Repositories" option on initial boot.
```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

### Snap 
```
sudo dnf install snapd
sudo ln -s /var/lib/snapd/snap /snap
sudo reboot now
sudo snap refresh
```

### GNOME volume step adjustment
```
gsettings set org.gnome.settings-daemon.plugins.media-keys volume-step 1
```

### Gnome Extensions and Tweaks
```
sudo dnf install gnome-tweaks
flatpak install flathub com.mattjakeman.ExtensionManager
```

### Firefox Theme
* https://github.com/rafaelmardojai/firefox-gnome-theme

### Grub Theme
* https://github.com/vinceliuice/grub2-themes

### Yaru Theme
sudo dnf install gnome-shell-theme-yaru
sudo dnf install yaru-theme

### Gnome Extensions
* Don't install these if you are using a different spin of Fedora.
* Pop Shell - run `sudo dnf install -y gnome-shell-extension-pop-shell xprop` to install it.
* [Quick Settings Tweaker](https://github.com/qwreey75/quick-settings-tweaks)
* [User Themes](https://extensions.gnome.org/extension/19/user-themes/)
* [Just Perfection](https://extensions.gnome.org/extension/3843/just-perfection/)
* [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)
* [Blur My Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/)
* [Vitals](https://extensions.gnome.org/extension/1460/vitals/)
* [Logo Menu](https://extensions.gnome.org/extension/4451/logo-menu/)
* [Space Bar](https://github.com/christopher-l/space-bar)

## Apps
* unzip
* p7zip
* unrar
* Blender
* Discord 
* Spotify
* GIMP
* Handbrake
* Krita
* Transmission
* VLC
* Darktable
* Rawtherapee
* mc
* bpytop
* inxi
* Libreoffice
* Audacity
