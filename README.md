# Arch Linux tips

Changelog and customization tips for my Arch Linux system, which is running on a Dell Precision 5530 laptop (same machine as the [XPS 15 (9570)](https://wiki.archlinux.org/index.php/Dell_XPS_15_9570).) If nothing else, it's a record of things I tend to forget.

![](desktop_dirty.png)
*Obligatory "dirty" screenshot running an even more obligatory MNIST example. Also, I like GNOME. (So sue me.)*

<!-- TOC depthFrom:2 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Installation](#installation)
	- [UEFI prep](#uefi-prep)
	- [Tip: Use a distribution](#tip-use-a-distribution)
	- [Home folder on separate partition (after install)](#home-folder-on-separate-partition-after-install)
- [NVIDIA GPU](#nvidia-gpu)
- [Backup](#backup)
- [Data science setup](#data-science-setup)
	- [R](#r)
	- [conda (Python)](#conda-python)
	- [Julia](#julia)
	- [GPU-enabled deep-learning (TensorFlow, CUDA, etc.)](#gpu-enabled-deep-learning-tensorflow-cuda-etc)
- [Miscellaneous](#miscellaneous)
	- [HiDPI](#hidpi)
	- [Tilix](#Tilix)
	- [Printing](#printing)
	- [Wi-fi from Shell/TTY](#wi-fi-from-shelltty)
	- [Starting Gnome session from Shell/TTY](#starting-gnome-session-from-shelltty)
	- [Manually compile a package from source with edited PKGBUILD (Julia example)](#manually-compile-a-package-from-source-with-edited-pkgbuild-julia-example)
	- [Touchpad](#touchpad)
	- [Removing Antergos](#removing-antergos)

<!-- /TOC -->

##  1. <a name='Installation'></a>Installation

###  1.1. <a name='UEFIprep'></a>UEFI prep

Follow the [Arch wiki](https://wiki.archlinux.org/index.php/Dell_XPS_15_9560#UEFI). Basically (1) switch SATA mode from RAID to ACHI, (2) disable secure boot, and (3) change fastboot to "Through" in "Post behaviour". Not sure that this last step is needed. I also enabled legacy BIOS drivers, since my system was having trouble reading the live USB.

###  1.2. <a name='Tip:Useadistribution'></a>Tip: Use a distribution

You can, of course, build Arch from scratch. But not everyone ~~is a masochist~~ has time for that. I installed my Arch system using the ([sadly discontinued](https://github.com/grantmcdermott/arch-tips#removing-antergos)) Antergos project. Luckily, there are many more options available. I recommend either:

- [EndeavourOS](https://endeavouros.com/): Picking up where the super Antergos community left off. Comes with all of the benefits of a bleeding-edge Arch setup without the pain. This would be my first choice if I were starting anew.
- [Manjaro](https://manjaro.org/): Another good option that a lot of people swear by. (Note, this is actually a derivative of Arch with its own repositories, etc.)

###  1.3. <a name='Homefolderonseparatepartitionafterinstall'></a>Home folder on separate partition (after install)

I think this was an option on the original install media, but I somehow missed it. At any rate, creating this after the fact wasn't too hard. I first created a GParted Live USB (download the ISO image [here](https://gparted.org/liveusb.php) and flash with Etcher). This was a lot quicker than creating a live USB of an entire distro and I only needed to resize some partitions anyway. From here, there are various guides (e.g. [here](https://help.ubuntu.com/community/Partitioning/Home/Moving) and [here](https://www.maketecheasier.com/move-home-folder-ubuntu/)) and I just followed along. FWIW, keeping your home directory on a separate partition is probably safer and also makes [distro hopping](https://www.maketecheasier.com/switch-between-linux-distros-without-losing-data/) easier.

##  2. <a name='NVIDIAGPU'></a>NVIDIA GPU

My Dell Precision 5530 laptop comes with a hybrid graphics system comprised of two card: 1) an integrated Intel GPU (UHD 630) and 2) an NVIDIA Quadro P2000. After various steps and misteps trying to install the NVIDIA drivers, I finally got everything working thanks to [this outstanding guide](https://wiki.archlinux.org/index.php/Dell_XPS_15_9570#Manually_loading/unloading_NVIDIA_module) on the Arch wiki. (Which, in turn, is based on this [this community thread](https://bbs.archlinux.org/viewtopic.php?pid=1826641#p1826641).). Some high level remarks:

- ~~The NVIDIA GPU and drivers only appear to work well on the Xorg session. Wayland performance is still shaky, or not even supported AFAIK.~~ Scratch that. The GPU is working just fine on the Wayland session. (Admittedly, I'm using it primarily for computation, not gaming.)
- The way the setup works is that the discrete NVIDIA GPU is switched off by default when the computer boots up to save battery life, etc.
- To turn it on, I just need to run a simple shell script that I've saved to my home directory.

```sh
$ cd ~ ## Just emphasising the location
$ sudo bash enablegpu.sh ## 'sudo bash disablegpu.sh' to turn off. Or, just shut down.
$ nvidia-smi ## Optional: confirm that the NVIDIA card has been enabled
```

**Tip:** To monitor GPU use and performance (a la `htop`), run `$ watch -n 1 nvidia-smi`. Or, you can install the [gpustat](https://github.com/wookayin/gpustat) command line utility and ran `$ gpustat -i`.

With the NVIDIA chip working, it's now fairly straightforward to set up a GPU-enabled deep-learning environment (see [here](https://github.com/grantmcdermott/arch-tips#gpu-enabled-deep-learning-tensorflow-cuda-etc)).

PS &mdash; As noted above, my working NVIDIA setup only came after various misteps. And the "solution" to these misteps back then was basically just to keep the discrete GPU switched off at all times. Thankfully, the real solution was easy enough thanks to the Arch wiki guide. Still, for posterity and since I learned a lot from working through these steps, the old changelog is under the fold...

<details>
  <summary>Old NVIDIA changelog (click to expand) </summary>
   I initially tried to get CUDA support going by installing the `nvida` package from the Arch repositories... Which turned out to be a mistake! The system would boot up fine, but I was subsequently presented with a blank screen once I got passed the GRUB menu.

   **Solution:** Boot directly into the shell (i.e. TTY) and uninstall the nvidia package: Press "Ctr-Alt-F2" at the grub menu and then hit "e" to edit the selection. Look for the line starting with "linux" and add "3" (without the quotation marks) to the end of that line. F10 to exit and then you will be presented with the shell upon booting up. Enter your username, followed by your password. Finally, uninstall the nvidia package by typing `sudo pacman -Rs nvidia` and
   reboot as normal ("CTR-ALT-DEL").

   **Update:** After some package and system updates, I'm back to the post-login blank screen! Weirdly, starting GDM from TTY1 (see above) works fine, so this is my current workaround. Have left a question on the [Antergos forum](https://forum.antergos.com/topic/11077/blank-screen-after-log-in-nvidia-issue) about this.

   **Update 2:** Added "nouveau.modeset=0" to the [kernel boot parameters](https://wiki.archlinux.org/index.php/Kernel_parameters#GRUB) as per various online suggestions:
   ```sh
   $ sudo nano /etc/default/grub
   ```
   Add "nouveau.modeset=0" to the GRUB\_CMDLINE\_LINUX\_DEFAULT variable. Then CTL+X and "y" to save. Re-generate the grub.cfg file:
   ```sh
   $ sudo grub-mkconfig -o /boot/grub/grub.cfg
   ```

   This solves the log-in and hibernate problem... but only for Xorg. In other words, now my Wayland session(s) have disappeared!

   **Update 3:** Have tried various fixes in the interim, including removing KDE/Plasma entirely in case there were some sytem conflicts with Gnome. I also tried removing the folder `~/.config/gnome-session` as per [this thread](https://bbs.archlinux.org/viewtopic.php?pid=1708172#p1708172). Didn't work. In fact, it turns out that the issue of GDM not being able to recognize Wayland sessions is common. Here are some relevant threads: [1](https://bbs.archlinux.org/viewtopic.php?id=225477), [2](https://www.reddit.com/r/archlinux/comments/823ye9/wayland_with_gnome/), [3](https://www.reddit.com/r/archlinux/comments/89vkwq/gnomegdm_issue_no_wayland_session/), [4](https://www.reddit.com/r/archlinux/comments/9wycf1/wayland_option_gone_from_login_screen_gnomegdm/). Will read through these various threads and try different options.

   **Update 4:** Solved! Thanks to [this](https://www.reddit.com/r/archlinux/comments/9wycf1/wayland_option_gone_from_login_screen_gnomegdm/) suggestion, I needed to enable early KMS start. (Basically, my laptop is too fast for its own good.) The full solution then is to disable modesetting for the Nouveau driver (see **Update 2** above) and enable early KMS start for the integrated Intel GPU, by adding following Intel modules to `/etc/mkinitcpio.conf`:
   ```
   /etc/mkinitcpio.conf
   ---
   MODULES=(intel_agp i915)
   ```

   Once that's done, regenerate initramfs:
   ```sh
   $ sudo mkinitcpio -p linux
   ```

   Reboot and I can now log directly into Gnome Wayland from GDM.

</details>

##  3. <a name='Backup'></a>Backup

Very easy with rsync. See [this video](https://www.youtube.com/watch?v=oS5uH0mzMTg).

```sh
$ bash ## zsh doesn't work for some reason
$ sudo rsync -aAXv --delete --dry-run --exclude=/dev/* --exclude=/proc/* --exclude=/sys/* --exclude=/tmp/* --exclude=/run/* --exclude=/mnt/* --exclude=/media/* --exclude="swapfile" --exclude="lost+found" --exclude=".cache" --exclude=".VirtualBoxVMs" --exclude=".ecryptfs" / /run/media/grant/PrecisionBackup
```

##  4. <a name='Datasciencesetup'></a>Data science setup

I followed (most of) the tips on Patrick Schratz' [outstanding guide](https://pjs-web.de/post/arch-install-guide-for-r/). His setup is tailored to R and spatial analysis, which covers my major needs. Here is a list of things that I did in addition to that, including other languages and GPU setup.

###  4.1. <a name='R'></a>R

Again, see Patrick's [guide](https://pjs-web.de/post/arch-install-guide-for-r/#r) for general installation and optimization tips. However, I also made the following additional changes.

####  4.1.1. <a name='SetcommonRlibrarypath'></a>Set common *R* library path

Adapting [this](https://stackoverflow.com/questions/44861967/r-3-4-1-single-candle-personal-library-path-error-unable-to-create-na/44903158#44903158) SO post answer, I set a system wide library path as follows:

```sh
$ sudo groupadd rusers
$ sudo gpasswd -a grant rusers
$ cd /usr/lib/R ## Get location by typing ".libPaths()" in your R console
$ sudo chown grant:rusers -R R/
$ sudo chmod -R 775 R/
```

Once that's done, tell *R* to make this shared library path the default for your user, by adding it to your `~/.Renviron` file:

```sh
$ echo 'R_LIBS_USER=/usr/lib/R/library' >>  ~/.Renviron
```

####  4.1.2. <a name='CompileRpackagesinparallel'></a>Compile R packages in parallel

Reinstallation of R packages is already much quicker thanks to [ccache](https://pjs-web.de/post/arch-install-guide-for-r/#ccache). However, I also enabled parallel compilation of R packages to speed up first-time installation, as well as any further compiling that needs to be done.

```sh
$ echo 'options(Ncpus=parallel::detectCores())' >> ~/.Rprofile
```

####  4.1.3. <a name='ReinstallRpackagesafterOpenBLASupdate'></a>Reinstall R packages after OpenBLAS update

An optimised BLAS library like OpenBLAS or MKL yields significant speed improvements. One [downside](https://twitter.com/grant_mcdermott/status/1199830893981884416) is that upgrading OpenBLAS requires manual reinstallation of the linked R packages. Trying to load the **sf** or **lfe** packages for example, will prompt the following error message: `libopenblas.so.3: cannot open shared object file: No such file of directory`

(Reason: R itself loads the packages and so they are "hidden" from the OS. This means that the normal OS resolution of dynamic library loading after an update &mdash; via `$ ldconfig` and co. &mdash; doesn't work.)

It's possible to fix this problem through trial and error; i.e. simply reinstall any package that prompts the above loading error manually. However, a much better way is to do everything in one fell swoop with [this script](https://gist.github.com/mllg/b9c75ded211df7df58942c5d647b9c43) from [Michael Lang](https://twitter.com/michellangts/status/1199990600919064576).

###  4.2. <a name='condaPython'></a>conda (Python)

Following [Jake Vanderplas](https://jakevdp.github.io/PythonDataScienceHandbook/00.00-preface.html#Installation-Considerations), I opted for Miniconda instead of the full-blown Anaconda install.

####  4.2.1. <a name='Addcondatopathifchangingshellsafterinstallation'></a>Add conda to path (if changing shells after installation)

In most cases, your conda environment should automatically get added to your PATH. However I installed Miniconda3 using bash before switching over to the zsh shell. As a result, I had to add the Miniconda directory to the zsh PATH environment variable (see [here](https://stackoverflow.com/a/35246794)).

```sh
$ echo 'export PATH="/home/grant/miniconda3/bin:$PATH"' >> .zshrc
$ source ~/.zshrc ## Or you can just close and reopen the shell
```

####  4.2.2. <a name='condaenvironments'></a>conda environments

To activate conda environments, you first need to initialise this capability for your particular shell (zsh, fish, etc.). However, this has the undesirable effect of automatically activating the previous conda env whenever you open the shell, regardless of whether you want to use conda or not. Luckily there's a pretty simple [solution](https://stackoverflow.com/a/54560785/4115816):

```sh
$ conda init zsh ## Initialise. Then log out and then back in.
$ conda config --set auto_activate_base false ## Stop automatic activation.
```

Once that's done, it's easy to activate and switch between environments

```sh
$ conda info --envs ## list all environments
$ conda activate tf_gpu ## activate the "tf_gpu" environment
$ conda deactivate
```

####  4.2.3. <a name='Addchannels'></a>Add channels

Necessary, for example, when I wanted to add the Apache Arrow C++ module from Conda Forge.

```sh
$ conda config --add channels conda-forge
$ conda config --set channel_priority strict
```

###  4.3. <a name='Julia'></a>Julia

There's a distributed version of Julia via the Arch community repos, but I eventually switched to using the official Julia binaries instead as is [recommended](https://julialang.org/downloads/platform.html). To take the pain out managing subsquent versions and updates, I used the handy [JILL](https://github.com/abelsiqueira/jill) script:

```sh
$ sudo bash -ci "$(curl -fsSL https://raw.githubusercontent.com/abelsiqueira/jill/master/jill.sh)"
```

###  4.4. <a name='GPU-enableddeep-learningTensorFlowCUDAetc.'></a>GPU-enabled deep-learning (TensorFlow, CUDA, etc.)

This section presumes that you have enabled your discrete GPU and installed the NVIDIA drivers ([here](https://github.com/grantmcdermott/arch-tips#nvidia-gpu)). After that, you have a couple of options. I actually ended up installing several CUDA-enabled environments, since I have the disk space and this was easy enough to do. As you as wish.

####  4.4.1. <a name='conda'></a>conda

As per [this](https://towardsdatascience.com/tensorflow-gpu-installation-made-easy-use-conda-instead-of-pip-52e5249374bc) killer blog post, all you need is

```sh
$ conda create --name tf_gpu tensorflow-gpu
```

####  4.4.2. <a name='Archrepos'></a>Arch repos

Alternatively, you can also install the TensorFlow packages directly from the [official Arch repos](https://www.archlinux.org/packages/?sort=&q=tensorflow&maintainer=&flagged=). For the GPU-versions:

```sh
$ pac install tensorflow-cuda python-tensorflow-cuda
```

####  4.4.3. <a name='R-1'></a>R

Finally, you can go through the excellent [tensorflow and keras](https://tensorflow.rstudio.com/) packages built for R. These allow for various installation options depending on your setup, as well as multiple installations for different Python virtual environments.

For example, the default `install_keras(tensorflow = "gpu")` approach will download and install everything to a new virtual environment located at `~/.virtualenvs/r-reticulate`.

As another example, assume that you already created the "tf_gpu" conda environment described above. Then you obviously don't need to install the whole Python setup again. Instead, just tell R to interface with this existing environment as follows:

```r
library(keras)
use_condaenv("tf_gpu") ## Now build your DL model...
```

##  5. <a name='Miscellaneous'></a>Miscellaneous

###  5.1. <a name='HiDPI'></a>HiDPI

The [Arch wiki](https://wiki.archlinux.org/index.php/HiDPI) has the goods here. I've even added a few sections for things that I had to troubleshoot. (E.g. Gnome Shell text scaling for Xorg sessions, although I primarily use Wayland.) Here are some additional things beyond that:

####  5.1.1. <a name='RStudio'></a>RStudio

Annoyingly, I added a wiki section on RStudio HiDPI scaling that was removed by a mod for reasons that make [absolutely no sense](https://wiki.archlinux.org/index.php?title=HiDPI&diff=586566&oldid=586565). At any rate if your RStudio fonts are too big, try editing the RStudio desktop app so that it recognizes an appropriate QT_SCALE_FACTOR environment variable. (A scaling of 0.75 works well for me, but play around.) Open `/usr/share/applications/rstudio.desktop` with your preferred text editor as root. Then change the first line to:

```
Exec=env QT_SCALE_FACTOR=0.75 /usr/bin/rstudio-bin %F
```

Next time you launch RStudio, all of the fonts (including menu items) should now be correctly scaled.

####  5.1.2. <a name='Texstudio'></a>Texstudio

This one took a little bit of troubleshooting. Everything was way too big initially. To fix, first go to _Options > Configure TeXstudio > Adv. Editor > Hacks/Workarounds_. (Note: Make sure the "Show Advanced Options" box at the bottom of the configure panel is checked.) Uncheck _Try to automatically choose best display options_. Then, change _Render Mode_ to "Qt". Cick OK and close TeXstudio.

Once that's done, similarly to what we did for RStudio, open up the desktop app at `/usr/share/applications/texstudio.desktop` and add an appropriate scaling factor.

```
Exec=env QT_SCALE_FACTOR=0.5 texstudio %F
```

Re-open Texstudio and bath in the serenity of your correctly scaled workspace.

####  5.1.3. <a name='Linuxconsolefont'></a>Linux console font

Another thing I'll add explicitly here is how to change the default Linux Console font that appears when booting up (or when booting into TTY). First download the terminus fonts family:
```sh
$ pac install terminus-font
```
You can then see the set of available fonts by typing `ls /usr/share/kbd/console`. To temporarily test out a larger font, type
```sh
$ setfont ter-132n
```
(Type `showconsolefont` if you want to see a table of the font's glyphs and letters).

To set this font permanently, open `/etc/vconsole.conf` with nano and add
```
FONT=ter-132n
```

###  5.2. <a name='Tilix'></a>Tilix

I prefer [Tilix](https://gnunn1.github.io/tilix-web/) to the default Gnome terminal. The old way of changing the default terminal no longer works in the latest versions of Gnome (see [here](https://bbs.archlinux.org/viewtopic.php?id=246952)). But a simple solution that covers most use cases is to modify the "Ctrl+Alt+T" shortcut. Go to `Settings > Devices > Keyboard Shortcuts`. At the bottom, change the "Terminal" command entry to "tilix" (from ").

###  5.3. <a name='Printing'></a>Printing

My home printer had been found automatically at first. However, I couldn't find it after a while for some reason. Adding it manually was a pain, because I didn't have the correct permissions in CUPS. (Adding myself to the "cups" user group didn't work either.) I solved the problem by following [these instructions](https://kernelmastery.com/enable-regular-users-to-add-printers-to-cups/).

###  5.4. <a name='Wi-fifromShellTTY'></a>Wi-fi from Shell/TTY

Relevant to cases where you need to log into the Shell/TTY to fix some hanging/freeze problem (e.g. CUDA/NVIDIA above). The easiest way I've found is to use `nmcli` (command line version of network manger). To see the available SSIDs type
```sh
$ nmcli dev wifi
```
Then connect with
```sh
$ nmcli dev wifi connect SSID_NAME password SSID_PASSWORD
```

###  5.5. <a name='StartingGnomesessionfromShellTTY'></a>Starting Gnome session from Shell/TTY

Similar rationale to the above:
```sh
$ XDG_SESSION_TYPE=wayland dbus-run-session gnome-session
```

Alternatively, launch via GDM:
```sh
$ sudo systemctl start gdm
```

###  5.6. <a name='ManuallycompileapackagefromsourcewitheditedPKGBUILDJuliaexample'></a>Manually compile a package from source with edited PKGBUILD (Julia example)

I initally installed [Julia](https://julialang.org/) via the Arch community repos. However, a build problem cropped up when upgrading to Julia 1.2.0; see [bug report](https://bugs.archlinux.org/task/63536?project=5&string=julia) and [GitHub issue](https://github.com/JuliaLang/julia/issues/33038). I ultimately took this as a sign to install the official Julia binaries instead. (See [above](#julia).) However, this also seemed like a good chance to practice compiling a package from the community repos with an edited PKGBUILD. There's suprisingly little guidance about this online, although [this Manjaro forum thread](https://forum.manjaro.org/t/how-to-install-pkgbuild-file-downloaded-manually/69365/2) proved very helpful. My full steps as follows:

1. Create some folder to download the relevant files to.  I used `~/julia` but I don't think the location really matters.
```sh
$ mkdir ~/julia
$ cd ~/julia
```
2. Download/create the [PKBUILD and ancilliary files](https://git.archlinux.org/svntogit/community.git/tree/trunk?h=packages/julia) to this location. There may be a smart way to to this automatically, but I just created the files manually using nano+copy+paste.
3. Edit the PKGBUILD (and any other files) as needed.
4. Build the package. This will take a while. Note: Using regular `$ makepkg -si` gave me errors. So instead I used:
```sh
$ makepkg -g >> PKGBUILD
#$ makepkg ## Gives GPG verification error. Fix that below first:
$ gpg --recv-key 66E3C7DC03D6E495 ## Add GPG key
$ makepkg ## NB: Don't use sudo!
```
5. Install the **pkg.tar** file. Be careful not to confuse with any of the other .tar files lying around in the same directory.
```sh
$ sudo pacman -U julia-2:1.2.0-1-x86_64.pkg.tar
```
6. The updated version of Julia was now ready to go:
```sh
$ julia
```

###  5.7. <a name='Touchpad'></a>Touchpad

_Note: Only relevant for the Plasma DE, which I'm no longer using._ The KDE graphical touchpad settings (*System Settings > Input Devices > Touchpad*) didn't seem to last and kept reverting back to the default behaviour. So I [installed](https://wiki.archlinux.org/index.php/Libinput#Installation) `libinput` and then followed the final section of [this guide](https://www.dell.com/support/article/us/en/04/sln308258/precision-xps-ubuntu-general-touchpad-mouse-issue-fix?lang=en) (See Fig. 7) to get tapping, right-click two finger tap, etc. working.

###  5.8. <a name='RemovingAntergos'></a>Removing Antergos

Following the [resolution of the Antergos Project](https://antergos.com/blog/antergos-linux-project-ends/), I removed all (or, at least, most) of the residual Antergos libraries following [these](https://forum.antergos.com/topic/11878/antefree-gnome) [guides](https://forum.antergos.com/topic/11887/antefree-gnome-cleaning-from-aur). This leaves a pure Arch system.

In related news, [Endeavour OS](https://endeavouros.com/) has picked up where Antergos left off and looks really cool.
