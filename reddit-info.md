From https://www.reddit.com/r/ableton/comments/1k43moz/ableton_move_i_swapped_out_the_proprietary_sd/

I'm gonna leave this here in case anyone else wants to try it; it worked out fine, but it took a little elbow grease. My efforts aren't complete, but here's my first day trying to mod the Move. My first success was replacing the Ableton proprietary 64GB SD Card. To be continued . . .

🏁 Preface
I'd skip this if I were you: I use Max/RNBO to make audio/video projects and sometimes I use Live for classical composition. As I travel often, Push is out of the question, so I was hoping I'd get a little controller that could be used in Live, but also plugged into my MacBook via USBC to Max so I could stream audio and control my Max instruments. The effects and synths are excellent, but I'm not a fan of rigid, looping, grid-based sequencing. Unfortunately, the current version only fulfills half of my needs.

👀 Overview
Quick Note: I'm not going to do tech support for beginners, so if you bork it, the fix is on you. All that being said, it's not rocket surgery & ChatGPT should be able to get you out of a jam, no problem. It's very malleable and well-built hardware. You'll be fine. 🤞🏼

I used a Sandisk Extreme Pro 512GB card, but most A2/V30 cards should work. CM4 isn't as hungry as CM5, so you don't need the latest and greatest, but I'd recommend Sandisk Extreme Pro. At any time you feel nervous, replace the SD Card with the Ableton one and boot; see? Still works.

🛠️ What You'll Need
- A Phillips-head screwdriver with a #0 tip

- MicroSD card (Save your physical Ableton proprietary card and create an IMG of the original for safety)

- ApplePi-Baker (macOS) to clone/restore; I have no idea how to use Windows

- Access to Linux; I use Ubuntu running in Parallels on MacOS; You 100% need Linux

- GParted for resizing partitions

🎁 Opening the unit
Believe it or not, this doesn't void your warranty as they'd tear the glue when they opened it at the service center and they'd have to prove you did something to harm the unit when you opened it.
You bought it, it's yours; do whatever you want.

The steps are basically:

- Rip the feet off
- Unscrew the back
- Lift the Raspberry Pi CM4 off of the rails
- Under two WiFi wires is the SD Card; can't miss it
- Look at you, Mr, Ms, or Mrs Hacker

🆙 Upgrade Steps
1. Clone Original SD:

- Use ApplePi-Baker to back up the Ableton 64GB SD card as an image

2. Restore to new SD Card:

- Flash the cloned image to the new card using ApplePi-Baker

3. (Crucial) Rename System Volumes:

- Mount the new card on Ubuntu

- Run:

sudo e2label /dev/sdc2 volumeI

sudo e2label /dev/sdc3 volumeII

4. Expand Data Partition:

- Use GParted:

Resize partition labeled 'data' (usually /dev/sdc4) to use all remaining space

5. Insert Card & Boot:

- Put the SD into the Move

- Boot normally everything should work if labels are correct

🤓 TL;DR Nerd Steps
**Prep a New Card** (Recommended: SanDisk Extreme Pro 512GB or 1TB)

**Clone the original 64GB card** using `ApplePiBaker` or `dd`

**Resize Partition 4 (`data`)** with `gparted` or CLI

**Fix UUIDs** if needed:- `e2label` or `tune2fs` to match original labels- Update `/etc/fstab` if you want to override mount behavior

💾 System Architecture & Modding Notes
System Overview (That I've recognized so far)

1. Raspberry Pi Compute Module 4 (CM4)

Primary SoC: Broadcom BCM2711

Quad-core Cortex-A72 (ARM v8) 64-bit SoC @ 1.5GHz with 4GB of RAM

Integrated GPU: VideoCore VI

Acts as the brain of the Move

Connected via custom carrier board (likely integrates GPIO, USB, audio, and display IO)

2. Intel AX200 (Wi-Fi 6 + Bluetooth 5.1)

Full-size M.2 card

Provides wireless connectivity

Standard PCIe M.2 interface

🛠️ Note: This is a bit unusual on a Pi-based system — suggests a high-performance carrier board with PCIe lanes properly routed.

3. XMOS XUF216 (Likely)

Found in /opt/move/MoveXmosCli and MoveXmosPower

Multicore USB Audio/DSP chip, often used for ultra-low-latency interfaces

Likely used for:

USB Audio Class 2 functionality (multi-channel in/out)

Clocking + latency compensation

Interface with USB-C ports

XMOS chip is used in RME, Focusrite, Universal Audio devices

4. Microchip USB Hub Controller (TBD model)

External USB-C, USB-A port

Possibly debugging ports

Likely something like Microchip USB2514B or similar

5. EEPROM or Flash Chip

Stores bootloader or board configuration

Could be a small SOIC 8 chip labeled with a 25-series number (e.g., W25Q32)

📝 Notes:
- No internal eMMC; boots directly from external microSD

- GPIO exposed internally for USB, audio, LED, etc.

🩻 Operating System: Debian 12 Bookworm (64-bit ARM, Lite variant)
- No GUI desktop

- Optimized for audio + real-time sequencing

- Kernel uses low-latency audio settings

📂 File System Layout
MicroSD card contains **4 partitions**:

/dev/mmcblk0p1 - boot (FAT16, 24MB) → Pi bootloader files

/dev/mmcblk0p2 - volumeI (ext4, ~44MB) → Primary root (active OS)

/dev/mmcblk0p3 - volumeII (ext4, ~44MB) → Secondary root (for A/B updates)

/dev/mmcblk0p4 - data (ext4, ~remainder of card GB) → Sample storage, patches, metadata

> **Note**: `volumeI` and `volumeII` appear to be part of a dual-partition failsafe boot/update system. Only one is active at a time. `data` is where most custom work can happen safely.

📋 Boot Process
- **Boot Configuration**: Defined in `/boot/cmdline.txt`

- Root is set to `root=/dev/mmcblk0p2` (or `p3` when updated)

- **Init**: Uses **SysV-style init scripts**, *not* systemd

- `/etc/init.d/move` starts `MoveLauncher`

- `MoveLauncher` boots the main `Move` binary

- **Main Application**:

- `/opt/move/Move` is the central binary (handles UI, audio engine, sequencing)

- Appears to statically reference most filepaths

- Launches RNBO patches via `/opt/move/rnbo/bin/RNBOLauncher`

🎧 Audio Architecture
- **DSP Engine**: Based on RNBO?? It seems like everything is built in RNBO

- DSP graphs are generated from Max/RNBO and launched by `RNBOLauncher`

- Preset patches and instruments are precompiled

- **Sample/Waveform Content**: Located in `/opt/move/Dsp/Vector/Sprites`

- Over 100+ `.wav` files categorized into groups:

- `Vector_Sprite_Basics`, `Harmonics`, `Formant`, `Noise`, `Distortion`, etc.

- File names are **hardcoded** in `Move`, not dynamically scanned

- Replacing samples requires matching exact filenames as I couldn't add new ones; to be continued

📜 Content in /data Partition
Mounted as `/dev/mmcblk0p4`:

- Houses user-accessible content and persistent data

- **Effects Configuration**:

- JSON files like `Reverb.json`, `Echo.json`, and `Compressor.json` define DSP parameters

- Format: likely tied to RNBO or internal schema

- **Instruments Directory**:

- May contain user-installed or experimental patches in future firmware versions, TBD

- **Preset Cache?**

- No clear dynamic preset storage found yet

- Possibly stored in-memory or pre-baked in `Move`

> **Note**: `data` is the best target for user customization. You can safely write to it, expand it, and mount it externally without bricking the OS.

👩🏾‍🔧 System Services and Components
- `MoveLauncher`: Bootstraps main UI and DSP engine

- `MoveSentryRunProcessor`: Likely hardware watchdog/service manager

- `MoveContentInfo`: Possibly handles patch metadata, TBD

- `MoveDBusTestClient` and `SystemDBusService`: DBus messaging layer for internal modules

- `MoveWebService` and `HttpRoot`: Hosts internal web interface (for remote access or OTA?)

👮🏻‍♂️ Security Notes
- No secure boot or encryption detected

- All partitions writable with root access

- No signs of file signing or DRM on patches or samples

- Good candidate for modding, but backup everything first

👟 My Next Steps
- **Goal A**: Replace `MoveLauncher` or override it to boot custom RNBO instrument directly

- **Goal B**: Write custom `.wav` files and explore dynamic patch loading

- **Goal C**: Decompile or intercept `Move` binary to understand DSP routing and UI hooks

- **Goal D**: Map RNBO preset switching behavior using `preset-switching.json`

- **Goal E**: Bridge Move over USB-C to Max for full hybrid rigs

- **Goal F**: 8 multi-channel audio & MIDI running over USBC; I've seen some people on Discord claim it can't be done because "they know the chips". Watch me, junior.

👨🏻‍⚖️ Legal Notes
This information is provided for educational and personal use only.

No proprietary software or licensed content is redistributed.

Reverse engineering is performed solely on owned hardware for artistic and research purposes.
