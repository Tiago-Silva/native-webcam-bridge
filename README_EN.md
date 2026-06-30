# 🎥 Native Webcam bridge

A high-performance, low-latency GUI tool written in Rust to transform your Android device (like the Samsung Galaxy S23) into a professional zero-delay webcam for **OBS Studio**, **Zoom**, **Google Meet**, and other applications.

---

### 🌐 Language / Idioma
* 🇬🇧 **English**
* 🇧🇷 **[Versão em Português deste documento aqui](README.md)**

---

<p align="center">
  <img src="assets/img-01.png" alt="Virtual Webcam Tab" width="49%" />
  <img src="assets/img-02.png" alt="NDI Screen Mirroring Tab" width="49%" />
</p>

---

## 🚀 How to Download and Run (No Compilation Needed)

You **do not need to compile any code** to use the application! The GitHub Actions workflow automatically compiles ready-to-use binaries on every update.

### 1. Download the Application
Go to the **[Releases](https://github.com/Tiago-Silva/native-webcam-bridge/releases)** section on the right side of this page and download the latest files:

* 🪟 **Windows**: Download the `Native_Cam_bridge_win64.zip` file.
  1. Extract the `.zip` archive into a folder of your choice.
  2. Double-click `native-cam-client.exe` to launch the graphical interface.
  *(The NDI DLL and all necessary Android device helper binaries are pre-packaged to work with zero-config installation!)*

* 🐧 **Linux**: Download the `Native_Cam_bridge.AppImage` file.
  1. Grant execution permissions to the downloaded file:
     ```bash
     chmod +x Native_Cam_bridge.AppImage
     ```
  2. Double-click the `AppImage` file or run it via terminal to launch.

---

## ✨ Key Features

* **Dual Operation Mode**:
  1. **Virtual Webcam Mode (scrcpy + v4l2loopback)**: Streams the physical camera sensor directly to a system virtual loopback video device (`/dev/video*`). Real zero-latency, native 16:9 aspect ratio (no black bars), and minimal CPU consumption.
  2. **NDI Screen Mirror Mode (Custom)**: Runs a lightweight native agent on your Android device to capture the screen, performs status-bar and border cropping via pixels, and streams the clean video feed over the local network as a professional NDI source.
* **Smart H.265 (HEVC) Codec Selection**: Automatically detects if your PC's GPU supports hardware HEVC decoding in FFmpeg upon startup. If supported, it defaults to H.265 for ultra-lightweight high-definition streaming (up to 4K). Otherwise, it automatically falls back to H.264.
* **Kernel Automation (Linux)**: Launches a graphical authentication dialog (`pkexec`) to load the `v4l2loopback` kernel module in real time.
* **Crash Prevention (Driver Reset)**: Features a **Reset/Remove Device** button in the GUI to instantly clear loopback video buffers and resolve resolution conflicts or green screens.
* **ADB USB & Wi-Fi Support**: Automatically detects connected devices and enforces ADB port forwarding (`--force-adb-forward`) to prevent reverse connection failures over wireless networks.

---

## 📐 Architecture Diagram

```mermaid
graph TD
    subgraph S23 / Android Device
        A[Camera Sensor] -->|scrcpy agent| B[MediaCodec Hardware Encoder]
        C[System View] -->|Native Cam Server| B
    end

    subgraph Host Computer (Native Webcam bridge Client)
        B -->|ADB USB / Wi-Fi Tunnel| D[Rust Client GUI]
        D -->|FFmpeg Demux / Decode| E{Selected Mode}
        E -->|Webcam Mode| F[v4l2loopback /dev/video9]
        E -->|NDI Mode| G[NDI SDK Sender]
    end

    F -->|V4L2 Video Device| H[OBS Studio / Zoom / Meet]
    G -->|Network NDI Source| H
```

---

## 📱 Android Phone Setup

1. On your phone, go to **Settings > About phone > Software information** and tap **Build number** 7 times to enable Developer Options.
2. Go back to the main settings page and enter **Developer Options**.
3. Enable **USB Debugging**.
4. *(Optional)* If you wish to connect wirelessly, enable **Wireless Debugging** as well and ensure your phone is connected to the same Wi-Fi network as your computer.

---

## 🔍 Troubleshooting

### 1. Camera Sensor Busy (Samsung Camera Eviction)
On Android, only one application can access the camera hardware sensor at a time.
* **Symptom**: `CameraAccessException` error.
* **Solution**: Fully close the native Samsung Camera app (or previews in Instagram/WhatsApp) on your phone. The Rust client opens the hardware sensor in the background, so no camera app needs to be running on the screen of your device.

### 2. Video Delay / Lag in OBS Studio
* **Symptom**: The video feed accumulates lag over time.
* **Solution**: In OBS Studio, double-click the "Video Capture Device (V4L2)" source corresponding to the camera and change **Buffering** to **Disable**. This forces OBS to render each frame instantly as it arrives in real time.

### 3. Green Screen in OBS when Changing Resolutions
* **Symptom**: Switching resolutions (e.g., from 1080p to 4K) results in a static green screen.
* **Solution**: The virtual loopback device is stuck on the previous resolution buffer in the kernel.
  1. Click **Stop Webcam** in the GUI.
  2. Click **Reset/Remove Device** in the app.
  3. Change the resolution in the dropdown and click **Create Device**.
  4. Start the stream by clicking **Start Webcam**.

---

## 📄 License

This project is open-source and distributed under the MIT License.
