---
title: "Fixing Audio on Dell 16 Plus with Ubuntu: Intel Lunar Lake RT722 Codec"
date: 2025-11-02T14:45:00-05:00
draft: false
tags:
  - Linux
  - Ubuntu
  - Audio
  - Hardware
  - Dell
---

If you've just installed Ubuntu on a Dell 16 Plus and found yourself staring at "Dummy Output" in your sound settings, you're not alone. The culprit? Intel's brand new Lunar Lake audio hardware with the RT722 codec - hardware so cutting-edge that Ubuntu's audio stack doesn't quite know what to do with it yet.

## The Problem

After a fresh Ubuntu installation on the Dell 16 Plus, the system shows:
- "Dummy Output" as the only audio device
- No speakers or headphones detected
- `wpctl status` showing no real audio sinks

Under the hood, the hardware is detected (`aplay -l` shows devices), but PipeWire/WirePlumber can't create usable audio endpoints because:

1. **The RT722 codec is too new** - UCM (Use Case Manager) profiles exist but fail during initialization
2. **Wrong profile selection** - WirePlumber defaults to "off" profile when the UCM fails
3. **Hardware muted** - The RT722 has internal speaker switches that default to off

## The Solution

The fix involves three components working together:

### 1. Use the Pro-Audio Profile

Instead of the standard UCM profiles (which fail), we use the "pro-audio" profile that exposes raw ALSA devices:

```bash
pactl set-card-profile alsa_card.pci-0000_00_1f.3-platform-sof_sdw pro-audio
```

### 2. Unmute the Hardware Switches

The RT722 codec has hardware switches that must be turned on:

```bash
amixer -c 0 set Speaker on
amixer -c 0 set Headphone on
```

### 3. Automate on Boot

Create a script at `~/bin/fix-audio.sh`:

```bash
#!/bin/bash
sleep 5

# Set pro-audio profile
pactl set-card-profile alsa_card.pci-0000_00_1f.3-platform-sof_sdw pro-audio 2>/dev/null
sleep 2

# Ensure speakers are on
amixer -c 0 set Speaker on 2>/dev/null
amixer -c 0 set Headphone on 2>/dev/null
```

Make it executable:
```bash
chmod +x ~/bin/fix-audio.sh
```

Create a systemd user service at `~/.config/systemd/user/fix-audio.service`:

```ini
[Unit]
Description=Fix Dell Audio
After=wireplumber.service
Requires=wireplumber.service

[Service]
Type=oneshot
ExecStart=/bin/bash %h/bin/fix-audio.sh
RemainAfterExit=yes

[Install]
WantedBy=default.target
```

Enable the service:
```bash
systemctl --user enable fix-audio.service
systemctl --user daemon-reload
```

## Limitations

This workaround has one quirk: **GNOME Settings won't show the audio devices** in its "Test Speakers" interface. This is because pro-audio mode exposes raw ALSA devices rather than the typical profiles GNOME expects. However, audio works perfectly fine - you just can't use the GUI test feature.

To test audio from the command line:
```bash
speaker-test -D pipewire -c 2 -t wav
```

## Why This Happens

Intel Lunar Lake is extremely new hardware (late 2024). The Sound Open Firmware (SOF) supports it, but the integration with ALSA's UCM system isn't fully mature yet. Specifically:

- The kernel driver loads correctly
- The topology files exist
- But UCM profile initialization fails when trying to prepare certain capture devices
- This causes the entire card profile to fail and default to "off"

> Future Ubuntu updates will likely fix this properly with updated UCM profiles. When that happens, you can simply disable the workaround service.

## Maintenance

Check if your workaround is running:
```bash
systemctl --user status fix-audio.service
```

After major updates, test if Ubuntu fixed it natively:
1. Disable the service temporarily
2. Reboot and check if audio works
3. If it works without the service, the fix is no longer needed!

```bash
systemctl --user disable fix-audio.service
```

## Conclusion

The hardware is fully supported at the kernel level - we just need a small workaround while the userspace audio stack catches up. This solution provides perfectly functional audio until Ubuntu's audio stack matures to handle the RT722 codec properly.

*This workaround was tested on Ubuntu 24.10 with kernel 6.14 on a Dell 16 Plus (2024 model) with Intel Lunar Lake processors.*