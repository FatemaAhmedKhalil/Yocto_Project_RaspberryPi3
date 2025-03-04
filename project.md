## Overview
This guide provides a step-by-step breakdown of The Project.

---

## BitBake 
1. **`build/conf/bblayers.conf`** → Specifies the layers included in the build, like `meta-openembedded` and `meta-yocto`.
2. **`build/conf/local.conf`** → Contains user-specific settings (e.g., `MACHINE`, `DISTRO`, `IMAGE_FSTYPES`).
3. **`meta/conf/layer.conf`** → Defines how BitBake processes each layer, including priorities and dependencies.

## BitBake Layers
- **`meta-poky`** → Provides the Poky reference distribution.
- **`meta-yocto-bsp`** → Contains Board Support Packages (BSPs) for reference hardware.
- **`meta-raspberrypi`** → Adds support for Raspberry Pi boards.
- **`meta-oe`** → Provides additional OpenEmbedded recipes and packages.
- **`meta-python`** → Includes extra Python packages.
- **`meta-networking`** → Provides networking utilities and services.
- **`meta-qt5`** → Adds support for the Qt5 framework.
- **Custom Layers:**
  - **`meta-info-distro`** → Infotainment distribution configurations.
  - **`meta-audio-distro`** → Audio distribution configurations.
  - **`meta-IVI`** → Contains an image recipe with a C++ application and Nano editor.

---

## Setting the Environment
### Install Dependencies (Ubuntu)
```bash
sudo apt install build-essential chrpath cpio debianutils diffstat file gawk gcc git iputils-ping \
libacl1 liblz4-tool locales python3 python3-git python3-jinja2 python3-pexpect python3-pip \
python3-subunit socat texinfo unzip wget xz-utils zstd git inkscape locales make \
python3-saneyaml python3-sphinx-rtd-theme sphinx texlive-latex-extra
```
Verify that `en_US.utf8` locale is available:
```bash
locale --all-locales | grep en_US.utf8
```

---

## Download Poky
Poky is the reference build system for the Yocto Project.
```bash
git clone -b kirkstone https://github.com/yoctoproject/poky.git
cd poky
```

---

## Directory Structure
Recommended directory structure:
```
yocto
├── build-raspberrypi3-32   # Build directory for Raspberry Pi 3 (32-bit)
├── downloads               # Shared directory for downloaded source files
├── images                  # Output directory for generated images
├── poky                    # Poky source directory
```
- **`downloads`** → Prevents redundant downloads.
- **`images`** → Stores output images.

---

## Initialize the Build Environment
Run inside `poky`:
```bash
source oe-init-build-env build-raspberrypi3-32
```
This creates the `build` directory and sets up environment variables.

---

## Configure Local Build Settings
Edit `local.conf`:
```bash
MACHINE ?= "raspberrypi3"
DL_DIR ?= "${TOPDIR}/../downloads"
BB_NUMBER_THREADS = "${@bb.utils.cpu_count()//2}"
PARALLEL_MAKE = "-j 4"
```
- **`MACHINE`** → Specifies target hardware.
- **`DL_DIR`** → Shared download directory.
- **`BB_NUMBER_THREADS & PARALLEL_MAKE`** → Optimize build using multiple CPU cores.
- **`bb.utils.cpu_count()`** → BitBake utility function that returns the number of CPU cores available on the system.  
- **`//2`** → Python's integer division.  

---

## Infotainment Distro
### Create the Layer Directory Structure
```bash
mkdir -p meta-info-distro/conf/distro 
touch meta-info-distro/conf/distro/infotainment.conf
```
Edit `infotainment.conf`:
```bash
DISTRO="infotainment"
DISTRO_NAME="Bullet-infotainment"
DISTRO_VERSION="1.0"

MAINTAINER="fatemahmedkhalil@gmail.com"


# SDK Information.
SDK_VENDOR = "-bulletSDK"
SDK_VERSION = "${@d.getVar('DISTRO_VERSION').replace('snapshot-${METADATA_REVISION}', 'snapshot')}"
SDK_VERSION[vardepvalue] = "${SDK_VERSION}"

SDK_NAME = "${DISTRO}-${TCLIBC}-${SDKMACHINE}-${IMAGE_BASENAME}-${TUNE_PKGARCH}-${MACHINE}"
# Installation path --> can be changed to ${HOME}-${DISTRO}-${SDK_VERSION}
SDKPATHINSTALL = "/opt/${DISTRO}/${SDK_VERSION}" 

# Disribution Feature --> NOTE: used to add customize package (for package usage).

# infotainment --> INFOTAINMENT

INFOTAINMENT_DEFAULT_DISTRO_FEATURES = "largefile opengl ptest multiarch vulkan x11 bluez5 bluetooth wifi qt5 info"
INFOTAINMENT_DEFAULT_EXTRA_RDEPENDS = "packagegroup-core-boot"
INFOTAINMENT_DEFAULT_EXTRA_RRECOMMENDS = "kernel-module-af-packet"

# TODO: to be org.

DISTRO_FEATURES ?= "${DISTRO_FEATURES_DEFAULT} ${INFOTAINMENT_DEFAULT_DISTRO_FEATURES} userland"

# install systemd  as init manager 
require conf/distro/include/systemd.inc

# prefered version for packages.
PREFERRED_VERSION_linux-yocto ?= "5.15%"
PREFERRED_VERSION_linux-yocto-rt ?= "5.15%"


# Build System configuration.

LOCALCONF_VERSION="2"

# add poky sanity bbclass
INHERIT += "poky-sanity"

```
Update `local.conf` to use infotainment distro:
```bash
DISTRO ?= "infotainment"
```

---

## Enable Systemd for Infotainment Distribution
Poky uses `sysvinit` by default. Switch to `systemd`:

### Create Directory Structure
```bash
mkdir -p meta-IVI/conf/distro/include
touch meta-IVI/conf/distro/include/systemd.inc
```

### Configure Systemd
```bash
# install systemd  as init manager 
DISTRO_FEATURES:append = " systemd" 


# select systemd as init manager 
VIRTUAL-RUNTIME_init_manager = " systemd"
VIRTUAL-RUNTIME_initscripts = " systemd-compat-units"
# VIRTUAL-RUNTIME_dev_manager = " busybox-mdev"
```

---

## Audio Distro
### Create the Layer Directory Structure
```bash
mkdir -p meta-audio-distro/conf/distro
touch meta-audio-distro/conf/distro/audio.conf
```
Edit `audio.conf`:
```bash
DISTRO="audio"
DISTRO_NAME="Bullet-audio"
DISTRO_VERSION="1.0"

MAINTAINER="fatemahmedkhalil@gmail.com"


# SDK Information.
SDK_VENDOR = "-bulletSDK"
SDK_VERSION = "${@d.getVar('DISTRO_VERSION').replace('snapshot-${METADATA_REVISION}', 'snapshot')}"
SDK_VERSION[vardepvalue] = "${SDK_VERSION}"

SDK_NAME = "${DISTRO}-${TCLIBC}-${SDKMACHINE}-${IMAGE_BASENAME}-${TUNE_PKGARCH}-${MACHINE}"
# Installation path --> can be changed to ${HOME}-${DISTRO}-${SDK_VERSION}
SDKPATHINSTALL = "/opt/${DISTRO}/${SDK_VERSION}" 

# Disribution Feature --> NOTE: used to add customize package (for package usage).

# audio --> AUDIO

AUDIO_DEFAULT_DISTRO_FEATURES = "largefile opengl ptest multiarch vulkan bluez5 bluetooth wifi audio_only"
AUDIO_DEFAULT_EXTRA_RDEPENDS = "packagegroup-core-boot"
AUDIO_DEFAULT_EXTRA_RRECOMMENDS = "kernel-module-af-packet"

# TODO: to be org.

DISTRO_FEATURES ?= "${DISTRO_FEATURES_DEFAULT} ${AUDIO_DEFAULT_DISTRO_FEATURES} userland"


# prefered version for packages.
PREFERRED_VERSION_linux-yocto ?= "5.15%"
PREFERRED_VERSION_linux-yocto-rt ?= "5.15%"


# Build System configuration.

LOCALCONF_VERSION="2"

# add poky sanity bbclass
INHERIT += "poky-sanity"
```

---

## Image Recipe: `ivi-test-image.bb`
### Create Directory Structure
```bash
mkdir -p meta-IVI/recipes-core/images
touch meta-IVI/recipes-core/images/ivi-test-image.bb
```

### Define Image Recipe
```bash
# Base this image on rpi-test-image
require recipes-core/images/rpi-test-image.bb

# Include Local Variables
SUMMARY="IVI Testing Image That Include RPI Functions and helloworld Package Recipes"

inherit audio

### IMAGE_INSTALL ###
IMAGE_INSTALL:append=" helloworld openssh nano"
# if DISTRO = "infotainment"
IMAGE_INSTALL:append="${@bb.utils.contains("DISTRO_FEATURES", "info", "rpi-play", " ", d)}"

### IMAGE_FEATURES ###
##########################################################
## 1. IMAGE_INSTALL --> ssh                             ##
## 2. do_rootfs -->                                     ##
##    - allow root access through ssh                   ##
##    - access root through ssh using empty password    ##
##########################################################
IMAGE_FEATURES:append=" ssh-server-openssh"

### MACHINE_FEATURES ###
MACHINE_FEATURES:append=" bluetooth wifi alsa"
```
**Base Image** 
`require`: Defines the core structure of the image by inheriting from an existing base image `rpi-test-image`.

**Inheritance** 
`inherit`: Some images inherit special classes that modify their behavior 
```bash 
inherit audio
``` 

**Package Installation** 
`IMAGE_INSTALL`: Specifies additional software packages to be included in the image `nano`, `helloworld`, `helloworld`, `openssh`, `rpi-play`.

**Image Features** 
`IMAGE_FEATURES`: Defines additional capabilities like SSH, debugging tools, or package management `ssh-server-openssh`, `debug-tweaks`.


**Machine Features** 
`MACHINE_FEATURES`: Defines hardware-specific features available for the target machine `alsa`, `wifi`, `bluetooth`.

---

## Creating Cpp App Recipe `helloworld`
```bash
mkdir -p ./meta-IVI/recipes-native-cpp/helloworld
cd /meta-IVI/recipes-native-cpp/helloworld
recipetool create -o helloworld_1.0.bb https://github.com/embeddedlinuxworkshop/y_t1.git
```
After Generate the Recipe, Add Some Changes, the Final Recipe:
```bash 
# Recipe created by recipetool
# This is the basis of a recipe and may need further editing in order to be fully functional.
# (Feel free to remove these comments when editing.)

# TODO: 1. Decumentation Variables
SUMMARY		= "Example for Native C++ Application for Testing YOCTO"
DESCRIPTION	= "Example for Native C++ Application for Testing YOCTO. Provided by Bullet Guru"
HOMEPAGE	= "http://github.com/embeddedlinuxworkshop/y_t1"

# Unable to find any files that looked like license statements. Check the accompanying
# documentation and source headers and set LICENSE and LIC_FILES_CHKSUM accordingly.
#
# NOTE: LICENSE is being set to "CLOSED" to allow you to at least start building - if
# this is not accurate with respect to the licensing of the software being built (it
# will not be in most cases) you must specify the correct value before using this
# recipe for anything other than initial testing/development!

# TODO: 2. Licence Variables
LICENSE = "CLOSED"
LIC_FILES_CHKSUM = ""

# TODO: 3. Source Code Variables
SRC_URI = "git://github.com/embeddedlinuxworkshop/y_t1;protocol=http;branch=master"

# Modify these as desired
PV = "1.0+git${SRCPV}"
SRCREV = "49600e3cd69332f0e7b8103918446302457cd950"

S = "${WORKDIR}/git"

# TODO: 4. Tasks Excuted through the Build Engine
# NOTE: no Makefile found, unable to determine what needs to be done

APPLICATION = "hello"

do_compile () {
	# Specify compilation commands here
	
	# Compile Cross-Compiler (Compiler Target )
	$CXX "${S}"/main.cpp -o "${APPLICATION}"
}

do_install () {
	# Specify install commands here
	
	# 1. manipulate -> ${WORKDIR}/image
	# 2. Create Directory ${WORKDIR}/image/usr/bin
	install -d "${D}"/"${bindir}"

	#3. installing hello bin in Directory ${WORKDIR}/image/usr/bin 
	install -m 0755 "${APPLICATION}" "${D}"/"${bindir}"
}

# Ignore do_package_qa
do_package_qa[noexec]="1"
```
**do_compile ()**
This function is automatically called during the build process to compile source code.
- `${CXX}` → Uses the C++ compiler set by Yocto.

**do_install ()**
used for copying and setting file permissions.
- `install -d` → Creates the destination directory.
- `${D}` `${bindir}` → Installs the compiled binary into `/usr/bin/` inside the target filesystem.
- `install -m 0755` → Copies the file and sets permissions (rwxr-xr-x).

---

## Integrating Nano
```bash
mkdir -p ./meta-IVI/recipes-editors/nano
cd layers/meta-IVI/recipes-editors/nano
recipetool create -o nano_1.0.bb https://ftp.gnu.org/gnu/nano/nano-7.2.tar.xz
bitbake nano
```
Install dependencies required for building Nano
```bash 
sudo apt install autoconf automake autopoint gcc gettext git groff make pkg-config texinfo
```
Fetch and unpack the source code
```bash
bitbake -c fetch nano
bitbake -c unpack nano
```
Find the WORKDIR path:
```bash
bitbake -e nano | grep -i "^workdir="
```
Navigate to the WORKDIR path.

Run autogen.sh to generate the configure script:
```bash
./autogen.sh
bitbake nano
```

---

## Integrate Audio
Create the `classes/` directory
```bash
cd ./meta-IVI
mkdir -p classes
touch classes/audio.bbclass
```
Edit the class: 
```bash
IMAGE_INSTALL:append = " pavucontrol pulseaudio pulseaudio-module-dbus-protocol pulseaudio-server \
        pulseaudio-module-loopback pulseaudio-module-bluetooth-discover alsa-ucm-conf pulseaudio-module-bluetooth-policy alsa-topology-conf alsa-state alsa-lib alsa-tools \
        pulseaudio-module-bluez5-device pulseaudio-module-bluez5-discover alsa-utils alsa-plugins packagegroup-rpi-test can-utils net-tools gstreamer1.0 \
        iproute2 iputils qtbase-examples qtquickcontrols qtbase-plugins libsocketcan qtquickcontrols2 qtgraphicaleffects qtmultimedia qtserialbus qtquicktimeline \
        qtvirtualkeyboard bluez5 i2c-tools hostapd iptables"
```

---

## rpi-play for Iphone
