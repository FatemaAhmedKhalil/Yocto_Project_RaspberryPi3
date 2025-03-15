### Application Requirements
## Target Hardware:
- Raspberry Pi 3
## Packages:
- SSH: Secure Shell for remote access.
- WIFI addon: Support for wireless connectivity.
- Nano: Simple text editor.
- Meta-qt5: Qt5 layer for developing graphical applications.
- Community: VSOMEIP: Middleware for inter-process communication.
- RPI Play: For screen mirroring.
- Audio: Support for audio playback and recording.
- Native Hello Bullet Application: A sample application for testing.

**Collaboration Project:** Submit your layer on Open-Embedded as part of the Bullet AI project.

## Image:
- Image recipe.
## Kernel:
- Version: 5.15.x
## Distributions:
**Distribution 1:**
- Excludes Meta-qt5.
- Uses systemd as the init system.
  
**Distribution 2:**
- Includes Meta-qt5.
- Uses sysvinit as the init system.
