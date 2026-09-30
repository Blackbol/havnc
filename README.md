# HAVNC (Home Assistant Dashboard through VNC) - iPad & Legacy Tablet Edition

[![Home Assistant Add-on](https://img.shields.io/badge/Home%20Assistant-Add--on-blue.svg)](https://www.home-assistant.io/)
[![Platform](https://img.shields.io/badge/Platform-iPad%202%2F3%2F4%20%7C%20iOS%209%20--%2010.3.3-green.svg)](https://apple.com)
[![License](https://img.shields.io/badge/License-BSD--2--Clause-orange.svg)](LICENSE.md)

A specialized Home Assistant Add-on that renders modern Lovelace dashboards inside a lightweight server-side Chromium instance and streams it to legacy tablets (such as **iPad 2, iPad 3, iPad 4 on iOS 9.3.5 / 10.3.3**) via an optimized HTML5 **noVNC** web interface.

This fork combines the best improvements from the HAVNC community, resolving the critical memory leaks, window decoration offsets, authentication bottlenecks, and disconnection issues that previously prevented 24/7 wall panel operation.

---

## 🌟 Key Improvements in this Fork

### 1. 🛡️ Critical Memory Leak Fixes for iOS 9/10 Safari
Legacy WebKit engines on early iPads (iPad 2, 3, 4) have very strict memory limits and a fragile garbage collector. The original noVNC v0.5.1 suffered from multiple severe memory leaks:
* **WebSocket Receive Queue Leak (`websock.js`)**: Fixed `concat()` allocating new unbounded byte arrays on every frame. Reduced buffer limits (`_rQmax = 2000`) and implemented aggressive queue compaction.
* **Canvas & Image Object Accumulation (`display.js`)**: Explicitly cleans up base64 Image objects (`img.src = ''`, `img = null`) and blitting references immediately after rendering.
* **Closure & Timer Leaks (`rfb.js`, `input.js`)**: Pre-bounds event handlers and cleans up double-click / timeout handlers, preventing thousands of function closures from accumulating in RAM.
> **Result**: The dashboard can run **24/7 as a wall-mounted panel** without crashing Safari or triggering reload loops.

### 2. 🖥️ True Borderless Fullscreen (Kiosk Mode)
* Implemented an Openbox kiosk configuration (`config/openbox/rc.xml`) disabling all title bars, window frames, and borders (`<decor>no</decor>`).
* Forces Chromium to start at coordinate `(0, 0)` with the exact display resolution (`1024x768` for iPad 4).
* Eliminates the blue window titlebar and the bottom resize border glitch.

### 3. 🔄 Automatic Reconnection
* Added automatic reconnection logic with exponential backoff and jitter if the Wi-Fi connection drops.
* Persists connection state to `localStorage`, allowing smooth recovery when iOS Safari wakes up.

### 4. 🔑 Optional Authentication (Passwordless by Default)
* The original add-on forced a mandatory VNC password. This fork allows running **without a password** on trusted local networks (`-SecurityTypes=None`).
* If a password is provided in the configuration, VNC authentication is automatically activated.

### 5. 💾 Session & Cookie Persistence
* Mounts Chromium's profile into persistent storage (`/data/google-chrome`), ensuring your Home Assistant login and view preferences remain saved across add-on restarts.

### 6. 🌐 Clean HTTP / WebSockets
* Removed invalid SSL cert references (`--cert / --key`) in WebSockify that caused handshake failures on legacy iOS Safari.

---

## 📐 Display Resolution Reference

| Device | iOS Version | Aspect Ratio | Native Resolution | Recommended Setting |
| :--- | :--- | :--- | :--- | :--- |
| **iPad 4** | iOS 10.3.3 | 4:3 | 2048 x 1536 (Retina) | `1024x768` |
| **iPad 2 / iPad mini 1** | iOS 9.3.5 | 4:3 | 1024 x 768 | `1024x768` |
| **iPad 3** | iOS 9.3.5 | 4:3 | 2048 x 1536 (Retina) | `1024x768` |
| **Custom / Portrait** | Any | - | - | e.g. `768x1024` |

---

## 🚀 Installation Guide

### Method A: Local Home Assistant Add-on (Recommended)

1. **Copy this repository to your Home Assistant `/addons` directory:**
   * **Via SSH (Terminal & SSH Add-on):**
     ```bash
     cd /addons
     git clone https://github.com/Blackbol/havnc.git havnc
     ```
   * **Via Samba Share:**
     Connect to `\\<HOME_ASSISTANT_IP>\addons` and copy this folder as `havnc`.

2. **Reload Add-on Store:**
   * In Home Assistant, go to **Settings** > **Add-ons** > **Add-on Store**.
   * Click the **⋮ (three dots)** menu in the upper right corner > **Check for updates**.
   * Refresh the page (F5). **HAVNC** will appear under **Local add-ons**.

3. **Install & Configure:**
   * Click on **HAVNC** and select **Install**.
   * Once installed, switch to the **Configuration** tab:
     ```yaml
     url: "http://homeassistant:8123/tableau-de-bord-ipad/0"
     resolution: "1024x768"
     password: ""
     ```
   * Ensure port **`8080`** is enabled in the Network section.
   * Toggle **Start on boot** and click **Start**.

---

## 📱 Setting Up Your iPad (Fullscreen Kiosk Mode)

1. On the iPad, open **Safari** and navigate to:
   ```text
   http://<HOME_ASSISTANT_IP>:8080
   ```
2. Once the dashboard loads, tap the **Share** button (box with an arrow pointing up).
3. Select **"Add to Home Screen"** (*Sur l'écran d'accueil*).
4. Launch the newly created icon from your Home Screen:
   * It will launch in **standalone web app mode**, removing the Safari URL bar and navigation buttons for a full-screen kiosk display!

---

## ⚙️ Configuration Reference

| Option | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `url` | string | `http://homeassistant:8123` | The target URL to open in Chromium (dashboard, specific view, etc.). |
| `resolution` | string | `1024x768` | The virtual Xvfb screen resolution (`WIDTHxHEIGHT`). |
| `password` | string | `""` | Optional VNC password. Leave empty for direct passwordless access on local networks. |

---

## 🛠️ Debugging & Memory Monitoring

For troubleshooting legacy WebKit sessions:
* `index-debug.html` includes real-time telemetry logging to the browser console.
* Detailed memory audit documentation is available in [`novnc-patches/DEBUGGING_GUIDE.md`](novnc-patches/DEBUGGING_GUIDE.md) and [`novnc-patches/MEMORY_LEAK_FIXES.md`](novnc-patches/MEMORY_LEAK_FIXES.md).

---

## 🤝 Credits & Acknowledgements

* **[gnyman/havnc](https://github.com/gnyman/havnc)**: Original project concept and Home Assistant add-on structure.
* **[erichelfenstens/havnc](https://github.com/erichelfenstens/havnc)**: Groundbreaking investigation and patches for noVNC memory leaks on iOS 9/10 WebKit.
* **[Joannou1/havnc](https://github.com/Joannou1/havnc)**: Auto-reconnect strategies and resilience improvements.
* **[underscorejasiu/havnc](https://github.com/underscorejasiu/havnc)**: Optional password architecture and VNC launch scripting.
* **[roseckyj/havnc](https://github.com/roseckyj/havnc)**: WebSockify parameter cleanup.
* **[noVNC](https://github.com/novnc/noVNC)**: HTML5 VNC client.
