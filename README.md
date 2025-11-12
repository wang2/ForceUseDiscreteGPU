# macOS Force dGPU Script

This repository provides a lightweight way to **keep macOS using the discrete GPU (dGPU)** — specifically designed for Intel-based MacBook Pros (e.g. 2019 16") that have both integrated (Intel UHD) and discrete (AMD Radeon) graphics.

> ⚠️ macOS 13+ (Ventura / Sonoma / Tahoe) heavily restricts GPU control.  
> On these versions, this script ensures `gpuswitch = 0` (prefers dGPU),  
> but full hardware enforcement may still require disabling *Automatic Graphics Switching* or using an external display.

---

## 🧩 File Structure

```
mac-force-dgpu/
├── README.md
├── scripts/
│   ├── force_dgpu.sh
│   └── check_gpu
└── launchd/
    └── com.user.force_dgpu.plist
```

---

## ⚙️ Installation

### 1. Copy the scripts
```bash
sudo cp scripts/force_dgpu.sh /usr/local/bin/
sudo cp scripts/check_gpu /usr/local/bin/
sudo cp launchd/com.user.force_dgpu.plist /Library/LaunchDaemons/
sudo chmod +x /usr/local/bin/force_dgpu.sh /usr/local/bin/check_gpu
```

### 2. Load the launch daemon
```bash
sudo launchctl load /Library/LaunchDaemons/com.user.force_dgpu.plist
```

The daemon will:
- Run `force_dgpu.sh` once at startup/login  
- Run again every 5 minutes to ensure dGPU preference is applied

---

## 💻 Files Explained

### **scripts/force_dgpu.sh**
A minimal shell script that sets macOS power mode to prefer the discrete GPU.

```bash
#!/bin/zsh
/usr/bin/pmset -a gpuswitch 0
exit 0
```

---

### **launchd/com.user.force_dgpu.plist**
Defines a background job that runs the script periodically.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.force_dgpu</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/force_dgpu.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>StandardOutPath</key>
    <string>/tmp/force_dgpu.daemon.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/force_dgpu.daemon.err</string>
</dict>
</plist>
```

---

### **scripts/check_gpu**
A diagnostic helper to view which GPU currently drives your display.

```bash
#!/bin/zsh
/usr/sbin/system_profiler SPDisplaysDataType 2>/dev/null | awk "
/Chipset Model:/ {chip=\$0}
/Display Type:/ {
  if (chip != \"\") {
    print chip;
    print \$0;
    print \"\";
    chip=\"\";
  }
}"
```

Usage:
```bash
check_gpu
```
Output example:
```
Chipset Model: AMD Radeon Pro 5300M
Display Type: Built-In Retina LCD
```
→ means your internal display is using the discrete GPU.

---

## 🔍 Verify Setup

```bash
pmset -g | grep gpuswitch
# should show: gpuswitch 0

check_gpu
# should show: Built-In Retina LCD under Radeon Pro
```

If still shows Intel UHD:
- Disable “Automatic graphics switching” in System Settings → Battery
- Reboot with power adapter plugged in
- Or connect an external monitor (forces dGPU)

---

## 🧹 Uninstall

```bash
sudo launchctl unload /Library/LaunchDaemons/com.user.force_dgpu.plist
sudo rm /Library/LaunchDaemons/com.user.force_dgpu.plist
sudo rm /usr/local/bin/force_dgpu.sh /usr/local/bin/check_gpu
```

---

## ⚠️ Notes & Limitations

- Apple removed official dGPU control APIs after macOS 12; `pmset gpuswitch` may no longer fully force dGPU.  
- On macOS 26 “Tahoe” and later, the OS may override manual GPU settings for power efficiency.  
- The most reliable way to force dGPU:  
  1. Disable *Automatic Graphics Switching*  
  2. Keep power adapter plugged in  
  3. Or attach an external display (Thunderbolt / HDMI / DisplayPort)

---

## 🧠 Credits

Inspired by community fixes for hybrid MacBook Pros (2015–2019).  
Maintained and simplified for modern macOS environments by wang2.

---
