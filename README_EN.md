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

## 🚀 How to Download

You **do not need to compile any code** to use the application! The GitHub Actions workflow automatically compiles ready-to-use binaries on every update.

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

## 🛠️ Step-by-Step Usage Instructions

To successfully use the tool, follow these steps:

### Step 1: Enable Developer Mode and Debugging on the Phone
Before anything else, set up your phone by following the steps in the [Android Phone Setup](#-android-phone-setup) section below. You must enable:
* **USB Debugging** (for wired connections).
* **Wireless Debugging** (if you prefer to connect over Wi-Fi).

---

### Step 2: Connect the Phone to the Computer
Choose one of the two connection options below:

#### Option A: Connection via USB Cable (Recommended for lowest latency)
1. Connect your phone to the computer using a high-quality USB cable.
2. **Authorize Debugging on the Phone (Crucial!)**:
   * When you plug in the cable, look at your phone's screen. A pop-up prompt asking **"Allow USB debugging?"** will appear.
   * Check the option **"Always allow from this computer"** and tap **Allow**.
   * *Without this authorization, the computer will not be able to communicate with your phone.*

#### Option B: Connection via Wi-Fi (Wireless)
1. Make sure both your computer and phone are connected to the **same Wi-Fi network**.
2. On your phone, go to **Settings > Developer Options > Wireless Debugging** and toggle it on.
3. Tap the text **"Wireless Debugging"** to enter the details screen. There, you will find the **IP address and Port** (e.g., `192.168.1.50:39845`).
4. **Authorize Debugging**: If this is your first time connecting, a prompt may appear on your phone asking to allow wireless debugging on this network. Tap **Allow**.
5. Open the **Native Webcam Bridge** application on your computer and enter the **IP Address** and **Port** into the corresponding Wi-Fi connection fields.

---

### Step 3: Choose the Mode in the Application and Start
Once your phone is connected and authorized, run **Native Webcam Bridge** on your computer and select one of the operation modes:

* **Virtual Webcam Mode**:
  - If you are on Linux, click **Create Device** and enter your administrator password to load the virtual kernel module (`v4l2loopback`). This step is not required on Windows.
  - Select your preferred resolution and codec.
  - Click **Start Webcam** to begin the video stream.
  - In OBS Studio, Zoom, or Google Meet, select the corresponding virtual camera device.
* **NDI Screen Mirror Mode**:
  - Select your preferred resolution and codec.
  - Click **Start NDI** to start mirroring and broadcasting the video feed to your local network.
  - In OBS Studio (with the OBS-NDI plugin installed) or any other NDI-compatible software, add an NDI Source and select the feed originating from the application.

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
