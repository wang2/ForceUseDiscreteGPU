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
 "http://www.apple.com/DTDs
