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

## Setting Up the Environment
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

## Download and Configure Poky
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
- `**//2`** → Python's integer division.  

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

---
