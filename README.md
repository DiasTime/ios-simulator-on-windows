<div align="center">

<img src="assets/banner.svg" alt="iOS 26.5 Simulator on Windows" width="100%">

**English** | [Русский](README.ru.md)

[![Guide PDF](https://img.shields.io/badge/guide-PDF%20·%2023%20pages-0969da?style=flat-square)](docs/iphone-simulator-on-windows_en.pdf)
![Host](https://img.shields.io/badge/host-Windows%2010%2F11-1f2328?style=flat-square&logo=windows)
![VMware](https://img.shields.io/badge/VMware%20Workstation-26H1-1f2328?style=flat-square&logo=vmware)
![macOS](https://img.shields.io/badge/macOS-Tahoe%2026-1f2328?style=flat-square&logo=apple)
![Xcode](https://img.shields.io/badge/Xcode-26.6-1f2328?style=flat-square&logo=xcode)
![iOS](https://img.shields.io/badge/iOS%20Simulator-26.5-1f2328?style=flat-square&logo=ios)
[![License](https://img.shields.io/badge/license-GPL--3.0-1f2328?style=flat-square)](LICENSE)

A step-by-step guide to running **macOS Tahoe** in **VMware Workstation**, installing **Xcode 26.6** and launching the **iOS 26.5 Simulator**, the newest iOS version you can get on a PC with an Intel CPU.

[**Download the guide (PDF)**](docs/iphone-simulator-on-windows_en.pdf) · [Русская версия PDF](docs/iphone-simulator-on-windows_ru.pdf)

</div>

---

## Contents

- [Why iOS 26.5 and not iOS 27](#why-ios-265-and-not-ios-27)
- [Requirements](#requirements)
- [Software versions](#software-versions)
- [Process overview](#process-overview)
- [Quick reference](#quick-reference)
- [Troubleshooting](#troubleshooting)
- [Rolling Windows back](#rolling-windows-back)
- [Limitations](#limitations)
- [Legal notice](#legal-notice)
- [Sources](#sources)

## Why iOS 26.5 and not iOS 27

Xcode 27 (released September 14, 2026) runs only on Apple Silicon Macs, and **macOS 26 Tahoe is the last macOS release for Intel**. A regular PC can only run the Intel build of macOS, so the ceiling is:

```
macOS Tahoe 26  →  Xcode 26.6  →  iOS 26.5 Simulator
```

This is the last combination available on x86 hardware at all.

| Plan | macOS in VM | Xcode | iOS Simulator | Stability in VM |
| :-- | :-- | :-- | :-- | :-- |
| **A** (main) | Tahoe 26.2+ | 26.6 | **26.5** | Works, limited support |
| **B** (fallback) | Sequoia 15.6+ | 26.3 | 26.2 | More stable, fewer mouse issues |

## Requirements

| | Minimum | Recommended |
| :-- | :-- | :-- |
| **CPU** | Intel with AVX2 (4th gen Core, Haswell, or newer) | Intel works best for macOS in VMware |
| **Virtualization** | Intel VT-x enabled in BIOS/UEFI | |
| **RAM** | 16 GB | 32 GB: 16 GB for the VM, 16 GB left for Windows |
| **Disk** | 100 GB free | 150 GB on an SSD (an HDD is very slow) |
| **OS** | Windows 10 or 11, 64-bit | Administrator rights are required |
| **Internet** | Stable connection, about 30 GB of traffic | macOS is downloaded from Apple during install |
| **Apple Account** | Free | Only needed to download Xcode |

> [!NOTE]
> Expect **2 to 4 hours** in total, most of it spent on downloads and the macOS install.

## Software versions

| Software | Version | Source |
| :-- | :-- | :-- |
| VMware Workstation Pro | 26H1 (or 26H1u1) | Broadcom support portal, free |
| Unlocker (BDisp) | 3.1.4 | [github.com/BDisp/unlocker](https://github.com/BDisp/unlocker/releases) |
| recoveryOS (DrDonk) | latest | [github.com/DrDonk/recoveryOS](https://github.com/DrDonk/recoveryOS/releases) |
| QEMU (`qemu-img` only) | latest | `winget install --id SoftwareFreedomConservancy.QEMU` |
| macOS Tahoe | 26.x | Downloaded from Apple during install |
| Xcode | 26.6, **Universal** build | [developer.apple.com/download/all](https://developer.apple.com/download/all) |
| iOS Simulator | 26.5 | Downloaded from inside Xcode |

## Process overview

| # | Stage | What happens | Time |
| :-: | :-- | :-- | :-- |
| 1 | **Prepare Windows** | Check VT-x, disable Hyper-V and Memory integrity | ~15 min + reboot |
| 2 | **VMware Workstation** | Download from Broadcom and install | ~15 min |
| 3 | **Unlocker** | Patch that adds macOS to the guest OS list | ~5 min |
| 4 | **Recovery image** | Download Tahoe recoveryOS from Apple, convert to VMDK | ~15 min |
| 5 | **Virtual machine** | Create the VM, attach the recovery disk | ~10 min |
| 6 | **Install macOS** | Partition the disk, network install, initial setup | 60-90 min |
| 7 | **Post-install** | VMware Tools, speed tweaks, shared folders | ~15 min |
| 8 | **Xcode and Simulator** | Xcode 26.6, iOS 26.5 platform, launch iPhone | 45-60 min |

The PDF covers every stage with annotated screens of each wizard and dialog.

## Quick reference

<details>
<summary><b>1. Disable Hyper-V</b> (PowerShell as Administrator)</summary>

```powershell
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V-All /NoRestart
dism.exe /Online /Disable-Feature:VirtualMachinePlatform /NoRestart
dism.exe /Online /Disable-Feature:HypervisorPlatform /NoRestart
bcdedit /set hypervisorlaunchtype off
```

Also turn off **Windows Security → Device security → Core isolation → Memory integrity**, then reboot.

> [!WARNING]
> WSL 2, Docker Desktop, Windows Sandbox and Hyper-V based Android emulators stop working after this. See [Rolling Windows back](#rolling-windows-back).

</details>

<details>
<summary><b>3. Apply Unlocker</b> (cmd as Administrator, VMware closed)</summary>

```bat
tasklist | findstr /I "vmware vmx"
cd /d "%USERPROFILE%\Downloads\unlocker-3.1.4"
win-install.cmd
```

Re-run `win-install.cmd` after **every** VMware update, since updates restore the original files.

</details>

<details>
<summary><b>4. Get the recovery image</b> (PowerShell)</summary>

```powershell
# Install qemu-img
winget install --id SoftwareFreedomConservancy.QEMU

# Manual download of Tahoe recovery and conversion to VMDK
.\macrecovery.exe -action download -board-id Mac-E1008331FDC96864 -mlb 00000000000000000 `
  -basename tahoe -outdir . -board-db .\boards.json
qemu-img convert -p -O vmdk -o compat6 tahoe.dmg tahoe.vmdk
```

</details>

<details>
<summary><b>5. VM settings</b></summary>

| Setting | Value |
| :-- | :-- |
| Configuration | Custom (advanced) |
| Guest OS | Apple Mac OS X → macOS 26 |
| Processors | **1** processor × **4** cores (even numbers only) |
| Memory | 16384 MB |
| Disk | SATA, 150 GB, single file, not preallocated |
| Second disk | Existing `tahoe.vmdk`, SATA, keep existing format |

Key lines in the `.vmx` file:

```ini
guestOS = "darwin25-64"        # Sequoia (plan B): "darwin24-64"
memsize = "16384"
numvcpus = "4"
cpuid.coresPerSocket = "4"
```

</details>

<details>
<summary><b>8. Xcode and Simulator</b> (macOS Terminal)</summary>

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -license accept
xcodebuild -runFirstLaunch

# iOS platform
xcodebuild -downloadPlatform iOS
xcrun simctl list runtimes

# Launch an iPhone and open a site
xcrun simctl boot "iPhone 17 Pro"
open -a Simulator
xcrun simctl openurl booted "https://example.com"
```

> [!IMPORTANT]
> Download the **Universal** Xcode build. The "Apple silicon" build will not run in the VM.

</details>

## Troubleshooting

| Problem | Fix |
| :-- | :-- |
| VT-x unavailable, VM won't power on | Enable Intel VT-x in BIOS, disable Hyper-V (stage 1) |
| Everything is very slow, `vmware.log` shows ULM monitor mode | VMware runs on top of Hyper-V. Repeat stage 1, check `msinfo32` |
| No Apple logo, boot loop, "The CPU has been disabled" | Re-apply Unlocker, match `guestOS` to the image, use 1 CPU × 4 cores |
| Tools installed but resolution and mouse are wrong | Click **Allow** in Privacy & Security, reboot macOS |
| "Xcode is not supported on this Mac" | You downloaded the Apple silicon build. Get **Universal** |
| iOS platform won't download in Xcode | `xcodebuild -downloadPlatform iOS` or import the `.dmg` manually |
| Unlocker doesn't work on your VMware version | Try [OC4VM](https://github.com/DrDonk/OC4VM), OpenCore based VM templates |

The full table is on pages 21-22 of the PDF, together with how to switch to **plan B (Sequoia)**.

## Rolling Windows back

```powershell
bcdedit /set hypervisorlaunchtype auto
dism.exe /Online /Enable-Feature:VirtualMachinePlatform /All /NoRestart
dism.exe /Online /Enable-Feature:HypervisorPlatform /All /NoRestart
dism.exe /Online /Enable-Feature:Microsoft-Hyper-V-All /All /NoRestart   # Pro/Enterprise only
```

Reboot afterwards and turn Memory integrity back on in Windows Security.

## Limitations

- **Apple Account inside macOS**: iCloud, App Store and system sign-in don't work. Signing in on developer.apple.com in a browser does, and that's enough for Xcode.
- **Which apps run**: only builds for the simulator (your Xcode project or a simulator `.app`). App Store apps and `.ipa` files for real devices won't run.
- **Speed**: no GPU acceleration, rendering is done by the CPU. Enough for testing sites and debugging your own app.
- **Liquid Glass**: Tahoe's translucency effects don't render in a VM. The simulator is unaffected.

## Legal notice

> [!CAUTION]
> The macOS license agreement allows installing macOS only on Apple hardware. Running it in a VM on a regular PC violates that agreement. Use this guide for personal learning and testing at your own risk. For commercial development, use a Mac or a cloud Mac (MacinCloud, MacStadium, AWS EC2 Mac).

## Sources

1. [Apple: Xcode system requirements](https://developer.apple.com/xcode/system-requirements/)
2. [Apple: Downloading and installing additional Xcode components](https://developer.apple.com/documentation/xcode/downloading-and-installing-additional-xcode-components)
3. [Xcode version history](https://mungomash.com/software/xcode/versions)
4. [Broadcom KB 368734: Download Desktop Hypervisor products](https://knowledge.broadcom.com/external/article/368734)
5. [macOS on VMware Workstation 26.x](https://github.com/aesiddiqui/macos-on-vmware-workstation)
6. [BDisp/unlocker](https://github.com/BDisp/unlocker)
7. [DrDonk/recoveryOS](https://github.com/DrDonk/recoveryOS)
8. [DrDonk/OC4VM wiki](https://github.com/DrDonk/OC4VM/wiki)

## License

[GPL-3.0](LICENSE)
