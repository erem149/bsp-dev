# BeagleBone Black Development Setup on Windows WSL2
## Complete Setup Guide with Troubleshooting Solutions

**Author:** ryankang63  
**Date:** January 2026  
**System:** Windows with WSL2 + Ubuntu 22.04 LTS  
**Target Hardware:** BeagleBone Black

---

## 📋 Table of Contents

1. [System Configuration](#system-configuration)
2. [WSL2 Installation](#wsl2-installation)
3. [Password Reset Solution](#password-reset-solution)
4. [Development Tools Installation](#development-tools-installation)
5. [USB Device Access Setup](#usb-device-access-setup)
6. [BeagleBone Hardware Connection](#beaglebone-hardware-connection)
7. [Serial Console Access](#serial-console-access)
8. [SD Card Management](#sd-card-management)
9. [Key Learnings](#key-learnings)
10. [Troubleshooting Reference](#troubleshooting-reference)
11. [Next Steps](#next-steps)

---

## System Configuration

- **Host OS:** Windows (with WSL2)
- **WSL Distribution:** Ubuntu 22.04 LTS
- **WSL Username:** ryankang63
- **Windows Username:** ASUS
- **Target Hardware:** BeagleBone Black
- **BeagleBone Connection:** Mini-USB (client port)
- **BUSID:** 5-2
- **Serial Device:** `/dev/ttyACM0`

---

## WSL2 Installation

### Install WSL2 with Ubuntu

```powershell
# In PowerShell (Administrator)
wsl --install -d Ubuntu-22.04
```

### First Launch Setup

1. Open Ubuntu from Start menu
2. Create username and password
3. **Write down your password!** You'll need it for sudo

### Verify Installation

```bash
# Check WSL version
wsl --list --verbose

# Check Ubuntu version
lsb_release -a
```

---

## Password Reset Solution

### Problem
- Forgot sudo password after WSL installation
- Multiple failed password attempts
- Cannot run any `sudo` commands

### Solution: Reset via Root Access

**From PowerShell (Admin):**

```powershell
# Login as root
wsl -u root
```

**Inside WSL as root:**

```bash
# Reset password for your user
passwd ryankang63

# Enter new password twice
# Exit root session
exit
```

**Test the new password:**

```bash
sudo apt update
# Enter your new password
```

### Important Notes

- ✅ **Universal method:** `wsl -u root` works on all WSL distributions
- ❌ **Avoid:** Distribution-specific commands like `ubuntu config --default-user root` (may not work)
- 💡 **Tip:** Use a memorable password or write it down securely

---

## Development Tools Installation

### Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### Core Build Tools

```bash
# Essential compilation tools
sudo apt install -y build-essential git gcc g++ make
```

### ARM Cross-Compilation Toolchain

```bash
# ARM cross-compiler for BeagleBone Black
sudo apt install -y gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf
```

### Kernel Build Dependencies

```bash
# Required for Linux kernel compilation
sudo apt install -y libncurses5-dev libssl-dev bc bison flex
```

### U-Boot Dependencies

```bash
# Bootloader development tools
sudo apt install -y device-tree-compiler u-boot-tools
```

### Disk Management Tools

```bash
# Filesystem and partition tools
sudo apt install -y fdisk parted dosfstools e2fsprogs
```

**Common typo to avoid:** It's `dosfstools` (with 's'), not `dosftools`

### Serial Console Utilities

```bash
# Terminal programs for serial access
sudo apt install -y minicom screen picocom
```

### Verify Installation

```bash
# Check cross-compiler
arm-linux-gnueabihf-gcc --version

# Check build tools
make --version
git --version
```

---

## USB Device Access Setup

### Install usbipd-win

**Problem:** `winget` command not available on some Windows versions

**Solution:** Manual installation

1. Download from: https://github.com/dorssel/usbipd-win/releases/latest
2. Download the `.msi` file (e.g., `usbipd-win_x.x.x.msi`)
3. Double-click to install
4. Click "Yes" when Windows asks for permission

### Verify Installation

```powershell
# In PowerShell (Admin)
usbipd --version
```

### Install WSL USB Tools

```bash
# In WSL Ubuntu
sudo apt install linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```

### Important Command Syntax Change

**Old syntax (deprecated):**
```powershell
usbipd wsl list          # ❌ No longer works
usbipd wsl attach        # ❌ No longer works
```

**New syntax (correct):**
```powershell
usbipd list              # ✅ Correct
usbipd attach --wsl      # ✅ Correct
```

---

## BeagleBone Hardware Connection

### Hardware Setup

**BeagleBone Black USB Ports:**

1. **Mini-USB Port (Client/Device)** ⭐ USE THIS
   - Location: Near Ethernet port
   - Provides: Power + Serial Console + USB Network
   - Connect this to your PC

2. **USB-A Port (Host)**
   - Location: Full-size USB port
   - Purpose: Connect USB devices TO the BeagleBone
   - Don't connect this to PC!

### Connection Method

**What you need:**
- Mini-USB to USB-A cable (like old phone charger)

**How to connect:**
1. Plug mini-USB end into BeagleBone Black
2. Plug USB-A end into your PC
3. Power LED on BeagleBone should light up

**No external 5V power needed!** USB bus power is sufficient for development.

### Critical Discovery: USB Port Issues

**Problem encountered:**
- Initially connected to USB port that didn't work
- Device not recognized in WSL
- No `/dev/ttyACM0` appeared

**Solution:**
- **Tried different USB port on PC**
- Device immediately recognized
- All functionality restored

**Key Learning:** If USB device not recognized, **try different USB ports first** before complex troubleshooting!

### Verify Connection in Windows

**Open Device Manager (Win + X → Device Manager):**

Look for:
- **Ports (COM & LPT):** USB Serial Device (COM3, COM4, etc.)
- **Network adapters:** Linux USB Ethernet/RNDIS Gadget

### Check USB Device in Windows

```powershell
# In PowerShell (Admin)
usbipd list
```

**Example output:**
```
BUSID  VID:PID    DEVICE
5-2    1d6b:0104  Remote NDIS Compatible Device, USB Serial Device...
```

**Your BeagleBone:** BUSID 5-2

---

## USB Device Attachment to WSL

### Attach BeagleBone to WSL

```powershell
# In PowerShell (Admin)

# Step 1: Bind the device (one-time setup)
usbipd bind --busid 5-2

# Step 2: Attach to WSL
usbipd attach --wsl --busid 5-2
```

### Verify in WSL

```bash
# Check USB devices
lsusb

# Check for serial device
ls /dev/ttyACM*
# Should show: /dev/ttyACM0

# List all block devices
lsblk
```

### Detach When Done

```powershell
# In PowerShell (Admin)
usbipd detach --busid 5-2
```

---

## Serial Console Access

### Method 1: Picocom (Recommended)

**Why recommended:**
- Simplest interface
- No multi-user issues
- Clean exit process

```bash
# Install
sudo apt install -y picocom

# Connect
sudo picocom -b 115200 /dev/ttyACM0

# Exit: Ctrl+A then Ctrl+X
```

### Method 2: Screen

```bash
# Connect
sudo screen /dev/ttyACM0 115200

# Exit: Ctrl+A then K then Y
```

**Problem encountered with screen:**
- Multiple "Screen used by root" password prompts
- Password loop that wouldn't accept input

**Solution:**
```bash
# Kill stuck screen sessions
sudo killall screen

# Or more forcefully
sudo pkill -9 screen

# Clean up dead sessions
screen -wipe
```

### Method 3: Minicom

```bash
# Install
sudo apt install -y minicom

# Connect
sudo minicom -D /dev/ttyACM0 -b 115200

# Exit: Ctrl+A then X then Enter
```

### BeagleBone Login Credentials

```
Username: debian
Password: temppwd
```

### Useful BeagleBone Commands

```bash
# After logging in:

# Check board model
cat /proc/device-tree/model

# Check kernel version
uname -a

# Check CPU info
cat /proc/cpuinfo

# Check memory
free -h

# Check storage
df -h

# Check OS version
cat /etc/os-release
```

### Alternative: Windows Serial Terminal

**Recommended for simplicity:**

**TeraTerm (Recommended):**
1. Download: https://ttssh2.osdn.jp/index.html.en
2. File → New Connection → Serial
3. Select COM port (from Device Manager)
4. Setup → Serial Port:
   - Speed: 115200
   - Data: 8 bit
   - Parity: none
   - Stop: 1 bit
   - Flow control: none

**PuTTY:**
1. Download: https://www.putty.org/
2. Connection type: Serial
3. Serial line: COM3 (your port)
4. Speed: 115200

---

## SD Card Management

### Problem: SD Card Locked by Windows

**Error when trying to mount:**
```powershell
wsl --mount \\.\PHYSICALDRIVE1 --bare
# Error: The disk is in use or locked by another process
```

### Solution: Take Disk Offline

#### Method 1: Disk Management (GUI - Easiest)

1. Press `Win + X` → Disk Management
2. Find your SD card (check size to identify)
3. Right-click on **disk number** (left side, e.g., "Disk 1")
4. Select **"Offline"**

#### Method 2: Diskpart (Command Line)

**Requires PowerShell/CMD as Administrator:**

```powershell
# Start diskpart
diskpart

# List all disks (verify SD card number by size!)
list disk

# Select SD card (VERIFY THIS IS CORRECT!)
select disk 1

# Take offline
offline disk

# Exit
exit
```

### Mount SD Card in WSL

```powershell
# In PowerShell (Admin)
# First verify disk number!
wmic diskdrive list brief

# Mount to WSL
wsl --mount \\.\PHYSICALDRIVE1 --bare
```

### Access SD Card in WSL

```bash
# Check available devices
lsblk

# Your SD card info from system:
# Device: /dev/sde (18M total, may need investigation if larger)
# Partition: /dev/sde1 (17M)

# Create mount point
sudo mkdir -p /mnt/sdcard

# Mount partition (NO TRAILING SLASHES!)
sudo mount /dev/sde1 /mnt/sdcard

# Check contents
ls -la /mnt/sdcard

# Navigate to it
cd /mnt/sdcard
```

### Important Syntax Note

**Common error:**
```bash
# ❌ WRONG - with trailing slashes
sudo mount /dev/sde1/ /mnt/sdcard/
# Error: special device /dev/sde1/ does not exist

# ✅ CORRECT - no trailing slashes
sudo mount /dev/sde1 /mnt/sdcard
```

### Unmount When Done

```bash
# Go back to home first
cd ~

# Unmount
sudo umount /mnt/sdcard
```

### Recommended Alternative: Balena Etcher

**For flashing BeagleBone images (much easier!):**

1. Download: https://etcher.balena.io/
2. Select image file
3. Select SD card
4. Click Flash
5. Done!

**Advantages:**
- ✅ No need to take disk offline
- ✅ Automatic verification
- ✅ User-friendly GUI
- ✅ Less error-prone

---

## Key Learnings

### 1. Command Syntax Precision

**Trailing slashes matter:**
- ❌ `sudo mount /dev/sde1/ /mnt/sdcard/`
- ✅ `sudo mount /dev/sde1 /mnt/sdcard`

**Exact package names:**
- ❌ `dosftools`
- ✅ `dosfstools`

**New usbipd syntax:**
- ❌ `usbipd wsl attach --busid X-X`
- ✅ `usbipd attach --wsl --busid X-X`

### 2. Hardware Troubleshooting Strategy

**USB port issues are common!**
- First solution: **Try different USB ports**
- Saved hours of complex troubleshooting
- Physical hardware often the simplest fix

**Connection verification:**
- Check Device Manager for COM ports
- Verify BUSID with `usbipd list`
- Check WSL with `lsblk` and `lsusb`

### 3. Tool Selection Wisdom

**Serial console:**
- `picocom` - Simplest, most reliable
- `screen` - Powerful but can have issues
- `minicom` - Feature-rich middle ground
- Windows TeraTerm - Often the easiest option

**SD card management:**
- Simple flashing: Balena Etcher (Windows)
- Advanced work: WSL mount after taking offline
- Don't overcomplicate what can be simple

### 4. WSL2 Hybrid Approach

**Best practice workflow:**
- ✅ WSL for building/compiling code
- ✅ Windows for hardware interfaces
- ✅ Combine strengths of both systems

**Why hybrid works:**
- Linux build environment is superior
- Windows hardware access is simpler
- No need to force everything through one system

### 5. Admin Privileges

**When you need Administrator:**
- PowerShell for usbipd commands
- Diskpart for disk management
- Installing software

**When you need sudo:**
- Mounting filesystems in WSL
- Installing packages with apt
- Accessing serial devices

**Password management:**
- Keep passwords documented securely
- `wsl -u root` for recovery access

---

## Troubleshooting Reference

### Quick Solutions Table

| Issue | Solution | Command/Action |
|-------|----------|----------------|
| Forgot sudo password | Use root to reset | `wsl -u root` then `passwd username` |
| USB device not showing | Try different USB port | Physical hardware check |
| usbipd command error | Update syntax | Use `attach --wsl` not `wsl attach` |
| Screen password loop | Kill stuck sessions | `sudo killall screen` |
| SD card locked | Take offline first | Disk Management → Right-click → Offline |
| Mount syntax error | Remove trailing slashes | `/dev/sde1` not `/dev/sde1/` |
| Package not found | Check spelling | `dosfstools` not `dosftools` |
| winget not available | Manual install | Download .msi from GitHub |
| Access denied in diskpart | Run as Administrator | Right-click → Run as administrator |
| Serial device not in WSL | Verify USB attachment | `lsusb` and `ls /dev/ttyACM*` |

### Diagnostic Commands

**Windows (PowerShell Admin):**
```powershell
# List USB devices
usbipd list

# Check disk drives
wmic diskdrive list brief

# List WSL distributions
wsl --list --verbose
```

**WSL Ubuntu:**
```bash
# Check USB devices
lsusb

# Check serial devices
ls /dev/ttyACM* /dev/ttyUSB*

# Check block devices
lsblk

# Check recent kernel messages
dmesg | tail -20

# Check mounted filesystems
df -h

# Verify cross-compiler
arm-linux-gnueabihf-gcc --version
```

---

## Directory Structure

### Recommended Workspace Organization

```
~/beaglebone-dev/
├── kernel/              # Linux kernel source
│   └── linux/          # Cloned from beagleboard/linux
├── u-boot/             # U-Boot bootloader
│   └── u-boot/         # Cloned from u-boot/u-boot
├── tools/              # Cross-compilation toolchain
│   └── gcc-arm-*/      # ARM toolchain
├── rootfs/             # Root filesystem work
├── projects/           # Your custom projects
└── setup-env.sh        # Environment setup script
```

### Environment Setup Script

Create `~/beaglebone-dev/setup-env.sh`:

```bash
#!/bin/bash

export BBB_WORKSPACE=~/beaglebone-dev
export PATH=$PATH:$BBB_WORKSPACE/tools/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf/bin

export ARCH=arm
export CROSS_COMPILE=arm-none-linux-gnueabihf-

export KERNEL_SRC=$BBB_WORKSPACE/kernel/linux
export UBOOT_SRC=$BBB_WORKSPACE/u-boot/u-boot

echo "BeagleBone Black development environment loaded!"
echo "ARCH=$ARCH"
echo "CROSS_COMPILE=$CROSS_COMPILE"
echo "Kernel source: $KERNEL_SRC"
echo "U-Boot source: $UBOOT_SRC"
```

**Make it executable:**
```bash
chmod +x ~/beaglebone-dev/setup-env.sh
```

**Use it:**
```bash
source ~/beaglebone-dev/setup-env.sh
```

---

## Next Steps

### Your environment is ready for:

1. **Download ARM Toolchain**
   - Official ARM toolchain or Linaro
   - Extract to `~/beaglebone-dev/tools/`

2. **Clone BeagleBone Kernel**
   ```bash
   cd ~/beaglebone-dev/kernel
   git clone https://github.com/beagleboard/linux.git
   cd linux
   git checkout 5.10  # LTS kernel
   ```

3. **Clone U-Boot**
   ```bash
   cd ~/beaglebone-dev/u-boot
   git clone https://github.com/u-boot/u-boot.git
   cd u-boot
   git checkout v2021.10
   ```

4. **Build First Kernel**
   ```bash
   cd ~/beaglebone-dev/kernel/linux
   make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig
   make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
   ```

5. **Study Device Trees**
   ```bash
   cd arch/arm/boot/dts
   ls am335x-bone*.dts
   ```

6. **Start BSP Projects**
   - Custom kernel modifications
   - Device tree overlays
   - Driver development
   - Bootloader customization

---

## Recommended Learning Path

### Phase 1: Foundation (2-3 weeks)
- Get comfortable with serial console
- Learn boot process (ROM → MLO → U-Boot → Kernel → Rootfs)
- Flash different distributions
- Explore existing device trees

### Phase 2: Kernel Development (3-4 weeks)
- Build kernel from source
- Modify device tree files
- Enable/disable kernel features
- Create simple device tree overlays

### Phase 3: Bootloader (2 weeks)
- Build U-Boot from source
- Modify boot environment
- Understand boot methods

### Phase 4: Driver Development (4-6 weeks)
- Write character device driver
- Create platform drivers
- Implement GPIO, I2C, SPI drivers

### Phase 5: BSP Creation (3-4 weeks)
- Learn Yocto or Buildroot
- Create custom layers
- Build complete BSP

---

## Resources

### Official Documentation
- BeagleBone Black System Reference Manual
- Texas Instruments AM335x Technical Reference Manual
- BeagleBoard.org: https://beagleboard.org/black
- Linux Device Drivers book (LDD3)
- Bootlin embedded Linux training (free!)

### Download Links
- BeagleBone images: https://www.beagleboard.org/distros
- TeraTerm: https://ttssh2.osdn.jp/index.html.en
- PuTTY: https://www.putty.org/
- Balena Etcher: https://etcher.balena.io/
- usbipd-win: https://github.com/dorssel/usbipd-win/releases
- ARM toolchain: https://developer.arm.com/downloads/-/gnu-a

### Community
- BeagleBoard forums
- Embedded Linux mailing lists
- GitHub: github.com/beagleboard

---

## Achievement Summary

### ✅ Completed Setup Tasks

- [x] WSL2 installation and configuration
- [x] Ubuntu 22.04 LTS setup
- [x] Password management and recovery
- [x] Development tools installation
- [x] Cross-compilation toolchain setup
- [x] USB device passthrough configuration
- [x] BeagleBone hardware connection
- [x] Serial console access (multiple methods)
- [x] SD card management workflow
- [x] Troubleshooting and optimization

### 🎯 Skills Acquired

- WSL2 administration
- USB device management in Windows/Linux
- Serial console communication
- Disk and partition management
- Linux command-line proficiency
- Hardware debugging techniques
- Cross-platform development workflow

### 🚀 Ready For

- BSP development
- Kernel compilation
- Device tree modification
- Driver development
- Embedded Linux customization

---

## Important Discoveries

### Critical Fixes That Made Everything Work

1. **USB Port Selection**
   - Single most important hardware fix
   - Different ports have different capabilities
   - Always test multiple ports

2. **Command Syntax Precision**
   - Trailing slashes break mount commands
   - Package names must be exact
   - Flag order matters in new usbipd syntax

3. **Root Access Method**
   - `wsl -u root` is universal and reliable
   - Don't rely on distribution-specific commands
   - Essential for password recovery

4. **Hybrid Development Approach**
   - Leverage strengths of both Windows and Linux
   - Don't force everything through one system
   - Use the right tool for each task

5. **Tool Alternatives**
   - Having multiple options (screen/minicom/picocom)
   - Windows tools often simpler for hardware
   - GUI tools reduce command-line errors

---

## Final Notes

This setup represents a complete, working BeagleBone Black development environment on Windows using WSL2. Every issue encountered has been documented with solutions, and every command has been tested.

The hybrid approach of using WSL2 for compilation and Windows for hardware interfaces provides the best of both worlds - powerful Linux development tools with straightforward Windows hardware access.

**You are now ready to begin serious BSP development!**

---

**Document Version:** 1.0  
**Last Updated:** January 17, 2026  
**Status:** Production Ready ✅
