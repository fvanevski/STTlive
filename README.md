# STTlive
# Project Design

Yes, that sounds like an excellent and well-thought-out plan! It correctly incorporates the key recommendations we've discussed and selects appropriate technologies for each component to meet your specific requirements for a high-performance, locally-run, and versatile live transcription tool.

Let's break down why this combination is strong:

1. **UI (Svelte):** A great choice for a reactive, performant user interface. Its compiler-based nature often leads to fast and lightweight UIs, which will be beneficial within the desktop and mobile shells.

2. **Windows Shell (Tauri):** Ideal for creating a desktop application from your Svelte UI. Tauri will allow you to:

1. Implement **global hotkeys** for activation.

2. Run the application in the **background** or from the system tray.

3. Create an **overlay** or "always-on-top" mini-UI.

4. It's generally more lightweight and secure than alternatives like Electron, leveraging the OS's native web rendering engine.

3. **Android Shell (Capacitor):** Perfect for taking your Svelte web application and packaging it as a native Android app. Capacitor will enable:

1. Integration with Nova Launcher gestures for app activation.

2. Running a **Foreground Service** for reliable background microphone access and transcription when you're using other apps.

3. Creating an **overlay UI** on Android.

4. Native **clipboard** access.

4. **Backend Core (RealtimeSTT-based):** This Python library is designed for real-time speech-to-text. Using it with faster-whisper as its engine will leverage your GPU effectively. It should handle aspects like Voice Activity Detection (VAD) and provide continuous transcription.

5. **Backend Environment (Conda on WSL 2 Ubuntu):** This is the optimal environment for running CUDA-accelerated Python workloads on your Windows machine.

1. WSL 2 provides near-native Linux performance and robust CUDA support.

2. Conda will manage the complex Python dependencies for AI/ML libraries, PyTorch, and CUDA toolkit versions.

6. **Hardware Acceleration (CUDA optimized packages on RTX 3090):** Absolutely essential for achieving the desired speed and accuracy from your local Whisper model via RealtimeSTT. Your RTX 3090 is well-suited for this.

**Key Interactions and Flow:**

1. Your Svelte UI, running inside Tauri (on Windows) or Capacitor (on Android), will capture microphone audio.

2. This audio will be streamed (likely via WebSockets) to your RealtimeSTT Python backend.

3. The Python backend, running in your CUDA-enabled WSL 2 Conda environment, will process the audio in real-time and send transcriptions back to the UI via WebSockets.

4. Tauri and Capacitor will provide the OS-level integrations (hotkeys, background services, overlay, clipboard, gesture handling on Android).

**Things to keep in mind during development:**

1. **Networking:**

1. The Tauri app on Windows should be able to connect to the WebSocket server hosted by the Python backend in WSL 2 (usually via localhost:<port> or 127.0.0.1:<port> from the Windows side).

2. The Capacitor app on Android, if it needs to connect to the backend running on your Windows PC's WSL 2, will require your Android device and PC to be on the same local network. You'll use the PC's local IP address (e.g., ws://192.168.x.x:<port>) in the Capacitor app, and you'll need to ensure your Windows Firewall and WSL 2 network configuration allow incoming connections to that port.

2. **Native Code in Capacitor:** Be prepared to write some native Java/Kotlin for Android to fully implement the Foreground Service for audio, the overlay UI, and potentially to handle specific intents from Nova Launcher gestures if you need more than just app launching.

3. **User Experience:** Think about how the user will manage settings, know the state of transcription (listening, paused, error), and interact with the overlay/mini-mode UIs.

Overall, your plan is solid, technologically sound, and aligns perfectly with the advanced features you're aiming for. It's an ambitious but very achievable project with this stack.

# Project Startup Guide

Okay, this is an ambitious and exciting project! You've chosen a powerful stack to bring it to life. This guide will provide step-by-step instructions for setting up your development environments, pointers to core documentation for coding the features you've discussed, and some additional considerations.

Given the complexity, each step will link you to official or highly relevant documentation where you can dive deeper. This guide aims to set you on the right path for each component.

**Current Date:** May 9, 2025

## ## Project Guide: AI-Powered Live Transcription Tool

**Core Technologies:**

1. **UI Framework:** Svelte (likely with SvelteKit)

2. **Windows Shell:** Tauri

3. **Android Shell:** Capacitor

4. **Backend Logic:** Python with RealtimeSTT (using faster-whisper for CUDA-accelerated transcription)

5. **Backend Environment:** Conda on WSL 2 (Ubuntu)

6. **GPU Acceleration:** NVIDIA RTX 3090 with CUDA

## ## Section 1: Backend Setup (Python with RealtimeSTT on WSL 2)

Your backend will handle the core speech-to-text processing.

### 1.1. WSL 2 and NVIDIA CUDA Driver Setup:

1. Ensure WSL 2 is installed and Ubuntu is set up:

1. Guide: [Install WSL on Windows](https://learn.microsoft.com/en-us/windows/wsl/install)

2. Install NVIDIA CUDA Driver for WSL 2: It's crucial to install the latest NVIDIA driver that supports WSL 2. This driver is installed on your Windows host.

1. Guide: [NVIDIA CUDA on WSL User Guide](https://docs.nvidia.com/cuda/wsl-user-guide/index.html) (Follow instructions for driver installation)

3. Verify GPU access in WSL 2: After driver installation, open your Ubuntu terminal in WSL 2 and run nvidia-smi. You should see your RTX 3090 listed.

### 1.2. Conda Environment Setup in WSL 2 (Ubuntu):

1. Install Miniconda or Anaconda in WSL 2 Ubuntu: If you don't have it, download and install it within your Ubuntu environment. Miniconda is recommended for a lighter setup.

1. Guide: [Installing Conda on Linux](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html) (Follow instructions within your WSL 2 Ubuntu terminal)

2. Create a Conda Environment:

1. Choose a Python Version: Python 3.9, 3.10, or 3.11 are generally good choices. RealtimeSTT and faster-whisper should be compatible. Let's aim for Python 3.10 as a balance.

2. Open your WSL 2 Ubuntu terminal.

3. Command: \
conda create -n live_transcribe_env python=3.10 \


4. Activate the environment: \
conda activate live_transcribe_env \


5. Conda Options/Channels: For GPU packages, especially PyTorch (a common dependency for Whisper-based tools), specific channels are often needed. You'll typically use these when installing PyTorch: -c pytorch -c nvidia -c conda-forge.

### 1.3. Install Core Backend Libraries:

1. PyTorch with CUDA: faster-whisper (used by RealtimeSTT) relies on CTranslate2, which often requires PyTorch's CUDA toolkit components to be correctly set up. Install PyTorch matching the CUDA version your NVIDIA driver supports and that faster-whisper expects (check their docs for specifics, but CUDA 11.8 or 12.1 are common as of early 2025).

1. Go to the [PyTorch Get Started page](https://pytorch.org/get-started/locally/)

2. Select: PyTorch Build (Stable), Your OS (Linux), Package (Conda), Language (Python), Compute Platform (CUDA 11.8 or CUDA 12.1 - choose one compatible with your driver and target libraries).

3. Example command (for CUDA 11.8, verify on PyTorch website for the latest command): \
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia \


4. (For CUDA 12.1, the command might be different, e.g., involving pytorch-cuda=12.1)

2. Install RealtimeSTT:

1. Check the official RealtimeSTT repository for the latest installation instructions and dependencies. It typically uses faster-whisper.

2. Likely installation via pip (once Conda environment is set up with PyTorch/CUDA): \
pip install RealtimeSTT faster-whisper \


3. RealtimeSTT GitHub (for reference and direct installation/usage instructions): Search "RealtimeSTT github" (e.g., a popular one is by KoljaB)

4. faster-whisper GitHub: [guillaumekln/faster-whisper](https://github.com/guillaumekln/faster-whisper) (Check for any specific CTranslate2/CUDA compilation notes if needed)

3. Install WebSocket Server Library: websockets or aiohttp are good choices for an asyncio-based Python WebSocket server. \
pip install websockets \


### 1.4. Verify Installation:

1. PyTorch CUDA: \
import torch \
print(f"PyTorch version: {torch.__version__}") \
print(f"CUDA available: {torch.cuda.is_available()}") \
if torch.cuda.is_available(): \
    print(f"CUDA version: {torch.version.cuda}") \
    print(f"GPU name: {torch.cuda.get_device_name(0)}") \


2. RealtimeSTT / faster-whisper: Try a basic transcription test using their command-line tools or a minimal script to ensure it utilizes the GPU. Refer to their respective documentation for example usage.

### 1.5. Coding the Backend Core Functions (Python with RealtimeSTT and WebSockets):

1. Real-time Transcription with RealtimeSTT:

1. You'll initialize RealtimeSTT specifying your desired Whisper model (which faster-whisper will run).

2. Feed audio chunks (received over WebSockets) into RealtimeSTT.

3. Get transcription results (interim and final) via callbacks or an iterable.

4. Documentation: Refer to the RealtimeSTT project's README and examples on GitHub.

2. WebSocket Server:

1. Create an asyncio WebSocket server that:

1. Accepts client connections (from your Svelte/Tauri/Capacitor app).

2. Receives binary audio data chunks.

3. Passes these chunks to RealtimeSTT.

4. Sends transcription results back to the connected client.

2. Documentation:

1. websockets library: [Python websockets library documentation](https://websockets.readthedocs.io/en/stable/)

2. aiohttp WebSockets: [aiohttp Server WebSockets](https://docs.aiohttp.org/en/stable/web_server.html#websockets)

## ## Section 2: UI Development (Svelte with SvelteKit)

SvelteKit is the official application framework for Svelte, providing routing, build tooling, etc.

### 2.1. Getting Started with Svelte/SvelteKit:

1. Node.js Installation: Ensure you have Node.js and npm installed. (Download from [nodejs.org](https://nodejs.org/))

2. Create a SvelteKit Project: \
npm create svelte@latest live-transcribe-ui \
cd live-transcribe-ui \
npm install \
npm run dev \


3. Official Documentation:

1. Svelte Tutorial: [Svelte Interactive Tutorial](https://svelte.dev/tutorial/basics)

2. SvelteKit Docs: [SvelteKit Documentation](https://kit.svelte.dev/docs)

### 2.2. Coding UI Core Functions (Svelte):

1. Microphone Access & Audio Streaming:

1. Use the Web Audio API (navigator.mediaDevices.getUserMedia, AudioContext, MediaStreamAudioSourceNode).

2. You might use an AudioWorkletNode or ScriptProcessorNode (older, runs on main thread) to get raw audio data (PCM) in chunks.

3. Send these chunks via a WebSocket connection to your Python backend.

4. Documentation:

1. Web Audio API (MDN): [MDN Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

2. getUserMedia (MDN): [MDN MediaDevices.getUserMedia()](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)

3. Svelte Stores for managing audio state: [Svelte Stores](https://svelte.dev/docs/svelte-store)

2. WebSocket Client:

1. Establish a WebSocket connection to your backend from the Svelte app.

2. Send audio data and receive transcription messages.

3. Update the UI with received transcriptions.

4. Documentation:

1. Browser WebSocket API (MDN): [MDN WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

2. Using WebSockets in Svelte: Integrate directly within component logic or services.

3. Displaying Transcriptions: Use Svelte's reactive declarations to update the text display.

4. Clipboard Copy Button:

1. Use the navigator.clipboard.writeText() API.

2. Documentation: [MDN Clipboard.writeText()](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText)

## ## Section 3: Windows Port (Tauri)

Tauri allows you to build a desktop app with your Svelte frontend.

### 3.1. Getting Started with Tauri:

1. Prerequisites: Install Rust, Node.js, and platform-specific build tools.

1. Guide: [Tauri Prerequisites](https://tauri.app/v1/guides/getting-started/prerequisites)

2. Integrate Tauri into your SvelteKit project:

1. Guide: [Tauri SvelteKit Integration](https://tauri.app/v1/guides/getting-started/setup/sveltekit) (Follow the steps to add Tauri to an existing SvelteKit project).

3. Official Documentation: [Tauri Documentation](https://tauri.app/v1/guides/)

### 3.2. Coding Windows-Specific Functions (Tauri):

1. Communication between Svelte frontend and Tauri backend (Rust or JS with tauri-api): [Tauri Inter-Process Communication (IPC)](https://tauri.app/v1/guides/features/ipc)

2. Global Hotkeys:

1. Use Tauri's globalShortcut module.

2. Documentation: [Tauri globalShortcut API](https://tauri.app/v1/api/js/modules/globalshortcut/)

3. Background Operation / System Tray:

1. Configure the app to run in the background.

2. Create a system tray icon with a menu.

3. Documentation:

1. [Tauri System Tray Guide](https://tauri.app/v1/guides/features/system-tray/)

2. [Tauri Window Behavior (e.g., hide on close)](https://tauri.app/v1/guides/features/window-customization)

4. Overlay / Always-on-Top UI:

1. Make the window always on top.

2. Potentially create a frameless, transparent window for a minimal overlay.

3. Documentation:

1. [Tauri Window.setAlwaysOnTop() API](https://tauri.app/v1/api/js/classes/window.web#setalwaysontop)

2. [Tauri Window Customization](https://tauri.app/v1/guides/features/window-customization)

5. Packaging your Svelte App: Tauri's build process handles this.

1. Documentation: [Tauri Building your App](https://tauri.app/v1/guides/building/introduction)

## ## Section 4: Android Port (Capacitor)

Capacitor wraps your Svelte web app into a native Android app.

### 4.1. Getting Started with Capacitor:

1. Prerequisites: Install Node.js, Android Studio, and necessary SDKs.

1. Guide: [Capacitor Environment Setup](https://capacitorjs.com/docs/getting-started/environment-setup)

2. Integrate Capacitor into your SvelteKit project:

1. Ensure your SvelteKit app is configured for Static Site Generation (SSG) using adapter-static if you're not using a custom dev server within Capacitor. Configure kit.paths.base if needed.

2. Guide: [Capacitor Installation](https://capacitorjs.com/docs/getting-started) (Follow steps to add Capacitor to an existing web project).

npm install @capacitor/core @capacitor/cli \
npx cap init [appName] [appId] --web-dir=build # Adjust web-dir if your SvelteKit output is different \
npm install @capacitor/android \
npx cap add android \
npx cap open android # Opens in Android Studio

3. Official Documentation: [Capacitor Docs](https://capacitorjs.com/docs)

4. SvelteKit with Capacitor: Search for community guides like "SvelteKit Capacitor" for specific integration tips, especially around the build output.

### 4.2. Coding Android-Specific Functions (Capacitor & Native Android):

1. Foreground Service for Background Audio & WebSocket:

1. This requires native Android code (Java/Kotlin). You'll create a service that runs in the foreground (with a persistent notification), accesses the microphone, and manages the WebSocket connection.

2. Communicate between your Svelte UI and this native service using Capacitor plugins or by creating your own custom plugin.

3. Documentation:

1. [Android Foreground Services](https://developer.android.com/guide/components/foreground-services)

2. [Capacitor Plugin Guide (for JS to Native communication)](https://capacitorjs.com/docs/plugins/creating-plugins)

4. Microphone access from Android Service: Standard Android AudioRecord API.

5. Managing WebSockets from a Java/Kotlin service: Use a Java WebSocket client library.

2. Overlay UI:

1. Request "Display over other apps" permission (SYSTEM_ALERT_WINDOW).

2. Create a floating UI using native Android views or a separate WebView.

3. Documentation:

1. [Android "Draw over other apps" permission](https://developer.android.com/reference/android/Manifest.permission#SYSTEM_ALERT_WINDOW)

2. [Android Floating Apps/Windows (various techniques)](https://developer.android.com/develop/ui/views/multi-window/picture-in-picture) (Picture-in-Picture might be adaptable, or custom window manager layouts).

4. Capacitor: You might need a custom plugin to manage this native overlay.

3. Clipboard Access:

1. Use the Capacitor Clipboard plugin or native Android clipboard API.

2. Documentation: [Capacitor Clipboard Plugin](https://capacitorjs.com/docs/apis/clipboard)

4. Handling Nova Launcher Gestures (Intents):

1. Configure your Android app (in AndroidManifest.xml and native code) to respond to specific intents if Nova Launcher can send them, or simply to resume/start a specific activity when launched.

2. Documentation:

1. [Android Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters)

2. Capacitor App API (for events like app state changes): [Capacitor App API](https://capacitorjs.com/docs/apis/app)

## ## Section 5: Did I Miss Anything? (Key Considerations)

Yes, a few things are important to keep in mind for a project of this scale:

1. **Error Handling & Resilience:** Implement robust error handling at every layer: UI, shells (Tauri/Capacitor), backend, and network communication. What happens if the mic isn't available, the WebSocket disconnects, the backend model fails to load, etc.?

2. **Configuration Management:**

1. How will users configure settings (e.g., Whisper model size, language, hotkeys, overlay preferences, auto-copy to clipboard)?

2. Consider simple JSON config files or UI-based settings screens. Tauri and Capacitor can help with storing/accessing these.

3. **State Management (UI):** For Svelte, [Svelte Stores](https://svelte.dev/docs/svelte-store) are excellent for managing application state (e.g., isTranscribing, currentTranscription, errorMessages).

4. **User Feedback & Logging:** Provide clear feedback to the user about the application's state. Implement logging (console, and perhaps to a file for the backend) for easier debugging.

5. **Packaging & Distribution:**

1. Tauri handles building installers/bundles for Windows.

2. Capacitor uses Android Studio to build APKs/AABs for Android.

3. Documentation: Covered in their respective "Building" or "Deployment" sections.

6. **Permissions:**

1. Windows: Generally fewer explicit permission hurdles for local apps unless accessing very sensitive APIs.

2. Android: Explicitly request permissions for Microphone, Display over other apps, and potentially others as needed at runtime. Capacitor can help manage this. [Capacitor Permissions (general concept)](https://capacitorjs.com/docs/guides/permissions), [Android Permissions](https://developer.android.com/guide/topics/permissions/overview).

7. **Performance and Resource Management:**

1. Live transcription is resource-intensive. Monitor CPU, GPU (for backend), and RAM usage.

2. Optimize audio processing pipelines.

3. Ensure smooth UI performance even while the backend is working hard.

8. **Security (Basic):** Since it's a local tool, major security risks are lower. However, be mindful if you ever expose the WebSocket server beyond localhost. Validate any input if applicable.

9. **Incremental Development & Testing:** Build and test each component incrementally. Get the backend working, then the basic Svelte UI with backend communication, then integrate with Tauri, then Capacitor. Automated tests, even simple ones, can be very helpful.

10. **User Experience (UX) for Overlay/Background Mode:** Carefully design how the user interacts with the tool when it's in a minimal or background state. How do they know it's working? How do they easily stop it or access its main UI?

This guide should give you a solid roadmap. Remember to consult the linked documentation frequently, as APIs and best practices can evolve. Good luck with your project – it sounds incredibly useful!

# Project PRDs

Yes, I can generate documents structured like Product Requirements Documents (PRDs) for each of the main modules of your project. These PRDs will be tailored to guide the development process, potentially with the assistance of an AI coding tool. They will outline the goals, features, technical specifications, and acceptance criteria for each component based on our detailed discussions.

**Important Caveats:**

1. **Guidance, Not Automation:** These PRDs are intended as detailed guides. While AI coding assistants can help accelerate development, they will still require significant human oversight, architectural decision-making, coding, debugging, and integration work.

2. **Based on Current Info:** These documents are based on the information we've discussed. Real-world PRDs often evolve with more stakeholder input, detailed UI/UX mockups (especially for frontend), and deeper iterative refinement.

3. **Focus on Functionality & Integration:** The PRDs will focus primarily on functionality, technical requirements, and API/component interactions, which are key for guiding an AI coding assistant. Aspects like detailed UI styling or highly specific algorithmic optimizations within RealtimeSTT (beyond using it as intended) would require further specification.

4. **Iterative Process:** Consider these PRDs as a starting point. You'll likely refine them as development progresses.

Here are the PRDs for the various modules of your AI-Powered Live Transcription Tool:

## ## PRD 1: Backend Transcription Service Module

1. **1. Project Module:** Backend Transcription Service

2. **2. Introduction & Goals:**

1. **Goal:** To create a high-performance, local, real-time speech-to-text (STT) service. This service will run on WSL 2, leverage an NVIDIA RTX 3090 GPU via CUDA-optimized packages, and use RealtimeSTT (with faster-whisper) as its core transcription engine. It will be accessible to client applications via a WebSocket interface.

2. **Purpose:** To provide the core, fast, and accurate transcription engine for the Svelte-based UI running in Tauri (Windows) and Capacitor (Android) shells.

3. **3. User Stories (Illustrative):**

1. As a client application, I want to stream raw audio data to the backend service so that I can receive real-time text transcriptions.

2. As a client application, I want to receive both interim (fast, possibly less accurate) and final (more accurate) transcription results to display to the user.

4. **4. Key Features & Functionality:**

1. **F1: STT Engine Initialization:**

1. Initialize RealtimeSTT with a pre-configured Whisper model (e.g., specified via a config file or environment variable) using faster-whisper for CUDA acceleration on the RTX 3090.

2. Ensure the model is loaded onto the GPU.

2. **F2: WebSocket Server:**

1. Implement an asyncio-based WebSocket server to handle client connections.

2. Listen on a configurable host and port (e.g., localhost:PORT within WSL 2, accessible from the Windows host).

3. **F3: Audio Data Ingestion:**

1. Accept raw audio data chunks (e.g., 16-bit PCM, mono, specified sample rate like 16000Hz) in binary format over the WebSocket from connected clients. Define expected audio format (e.g., sample rate, channels, bit depth).

4. **F4: Real-time Transcription Processing:**

1. Continuously feed incoming audio chunks to the initialized RealtimeSTT instance.

5. **F5: Transcription Data Streaming:**

1. As RealtimeSTT produces transcription results (interim and final), send these back to the originating client via the WebSocket connection.

2. Differentiate between interim and final results in the message payload.

6. **F6: Error Handling & Logging:**

1. Implement robust error handling for WebSocket connections, audio processing, and STT engine issues.

2. Log significant events and errors to the console (and optionally to a file).

3. Send error messages back to clients over WebSockets if appropriate.

7. **F7: Configuration:**

1. Allow configuration of the Whisper model to be used (e.g., tiny, base, small, medium).

2. Allow configuration of the WebSocket server port.

5. **5. Technical Requirements & Specifications:**

1. **Language:** Python (version 3.10 recommended).

2. **Core Libraries:** RealtimeSTT, faster-whisper, websockets (Python library), asyncio, PyTorch (with CUDA support).

3. **Environment:** Conda environment running on WSL 2 (Ubuntu).

4. **Hardware Target:** NVIDIA RTX 3090 with appropriate CUDA Toolkit and cuDNN versions compatible with PyTorch and faster-whisper.

5. **WebSocket Protocol:**

1. **Client to Server (Audio):** Binary messages containing raw audio chunks.

2. **Server to Client (Transcription/Status):** JSON messages.

1. *Example Transcription:* {"type": "interim", "text": "Hello world"}

2. *Example Transcription:* {"type": "final", "text": "Hello world."}

3. *Example Error:* {"type": "error", "message": "Transcription engine error"}

4. *Example Status (Optional):* {"type": "status", "status": "listening"}

6. **6. Acceptance Criteria:**

1. **AC1:** Backend service starts, initializes RealtimeSTT with the specified Whisper model, and successfully loads the model onto the GPU (verified via nvidia-smi in WSL 2 showing utilization).

2. **AC2:** WebSocket server starts and listens on the configured port, accepting connections from a test client.

3. **AC3:** When raw audio data is streamed from a test client, the backend processes it using RealtimeSTT.

4. **AC4:** Interim and final transcription results are accurately sent back to the test client via WebSockets in the defined JSON format.

5. **AC5:** The service handles client disconnections gracefully.

6. **AC6:** Basic error conditions (e.g., invalid audio format if checked) are handled and reported.

7. **7. Dependencies:**

1. Correctly configured Conda environment with all specified Python libraries.

2. Functional CUDA drivers and toolkit within WSL 2.

3. Access to Whisper model files.

8. **8. Out of Scope (Initially):**

1. Support for multiple simultaneous transcription languages (focus on one primary language first).

2. Advanced audio pre-processing beyond what RealtimeSTT provides.

3. User authentication/authorization for WebSocket connections (assuming a trusted local environment).

## ## PRD 2: Core UI Module (Svelte/SvelteKit)

1. **1. Project Module:** Core User Interface (Svelte/SvelteKit)

2. **2. Introduction & Goals:**

1. **Goal:** To develop a responsive and real-time web-based user interface using Svelte/SvelteKit. This UI will capture microphone audio, stream it to the backend transcription service, display incoming transcriptions, and provide basic user controls like copy to clipboard.

2. **Purpose:** To serve as the primary user-facing component, embedded within Tauri (Windows) and Capacitor (Android) shells.

3. **3. User Stories:**

1. As a user, I want to see the status of the transcription (e.g., idle, listening, error).

2. As a user, I want to see my spoken words transcribed in real-time (or near real-time).

3. As a user, I want to easily copy the full transcribed text to my clipboard.

4. As a user (implicitly, for shell integration), I want the UI to be controllable (start/stop) by the parent shell.

4. **4. Key Features & Functionality:**

1. **F1: Microphone Access & Audio Capture:**

1. Request microphone permission from the user.

2. Capture audio using the Web Audio API (navigator.mediaDevices.getUserMedia).

3. Process audio into chunks suitable for streaming (e.g., raw PCM Float32Array).

2. **F2: WebSocket Communication Client:**

1. Establish and maintain a WebSocket connection to the backend transcription service.

2. Send captured audio chunks to the backend.

3. Receive transcription messages (interim, final, error, status) from the backend.

3. **F3: Real-time Transcription Display:**

1. Display interim and final transcription results as they arrive from the backend.

2. Update the display efficiently.

4. **F4: UI State Management:**

1. Manage UI state (e.g., isConnecting, isListening, isError, currentTranscriptionText) using Svelte stores.

2. Display appropriate status indicators to the user.

5. **F5: Copy to Clipboard:**

1. Provide a button that, when clicked, copies the accumulated final transcription text to the user's clipboard (navigator.clipboard.writeText()).

6. **F6: Controllability Hooks (for Shells):**

1. Expose JavaScript functions or use Svelte store subscriptions that the parent shell (Tauri/Capacitor) can call/trigger to:

1. Start microphone capture and audio streaming.

2. Stop microphone capture and audio streaming.

3. Request current transcription text for clipboard (alternative to button).

7. **F7: Basic Error Display:** Show user-friendly messages if WebSocket connection fails or backend sends an error.

5. **5. Technical Requirements & Specifications:**

1. **Framework:** Svelte with SvelteKit.

2. **Language:** JavaScript/TypeScript.

3. **Key Web APIs:** Web Audio API, WebSocket API, Clipboard API.

4. **State Management:** Svelte Stores.

5. **Communication Protocol:** Adhere to the WebSocket message formats defined in the Backend PRD.

6. **Build Output:** Static web assets that can be embedded in Tauri and Capacitor. SvelteKit's adapter-static might be used.

6. **6. Acceptance Criteria:**

1. **AC1:** UI successfully requests and obtains microphone permission.

2. **AC2:** UI establishes a WebSocket connection to the backend service.

3. **AC3:** When the user speaks (and capture is active), audio is streamed to the backend.

4. **AC4:** Interim and final transcription text from the backend is displayed in real-time in the UI.

5. **AC5:** The "Copy to Clipboard" button correctly copies the displayed transcription.

6. **AC6:** UI correctly reflects different states (e.g., connecting, listening, idle, error).

7. **AC7:** Exposed JavaScript functions for starting/stopping transcription can be successfully called (tested in browser console initially).

7. **7. Dependencies:**

1. A running and accessible Backend Transcription Service.

2. Modern web browser for development and initial testing.

8. **8. Out of Scope (Initially):**

1. Advanced UI customization (themes, font choices beyond defaults).

2. Saving/loading transcription history within the UI itself.

3. User accounts or settings persistence beyond session (shells might handle some settings).

## ## PRD 3: Windows Shell Module (Tauri Integration)

1. **1. Project Module:** Windows Desktop Shell (Tauri)

2. **2. Introduction & Goals:**

1. **Goal:** To package the Svelte-based UI into a native Windows desktop application using Tauri. This shell will provide OS-level integrations like global hotkey activation, background/system tray operation, an optional overlay/always-on-top mode, and ensure seamless communication with the Svelte UI and the backend service.

2. **Purpose:** To provide a native-like, convenient user experience on Windows for the live transcription tool.

3. **3. User Stories:**

1. As a Windows user, I want to start/stop live transcription using a global keyboard shortcut, even when another application is active.

2. As a Windows user, I want the transcription app to run unobtrusively in the background or system tray.

3. As a Windows user, I want an option to see the live transcription in a small, always-on-top window while I work in other applications.

4. As a Windows user, I want to easily copy the transcription to my clipboard, possibly via a hotkey or a tray menu option.

4. **4. Key Features & Functionality:**

1. **F1: Svelte UI Embedding:**

1. Successfully embed and display the Svelte/SvelteKit web UI within a Tauri window.

2. **F2: Global Hotkey Activation:**

1. Register a configurable global hotkey (e.g., Ctrl+Shift+Space) to toggle the transcription state (start/stop).

2. When the hotkey is pressed, communicate with the embedded Svelte UI to trigger the start/stop transcription action.

3. **F3: Background Operation & System Tray:**

1. Allow the application to run in the background when the main window is closed or minimized.

2. Provide a system tray icon with a context menu (e.g., Start/Stop Transcription, Show Window, Copy Last Transcription, Quit).

4. **F4: Overlay / Always-on-Top Mode:**

1. Provide an option (e.g., via tray menu or a setting) to switch to a compact, always-on-top window mode for displaying transcriptions. This window might be frameless or have minimal decorations.

5. **F5: Clipboard Integration (Shell-level):**

1. Allow copying the latest transcription to the clipboard via a tray menu option or a secondary hotkey. This would involve getting the current transcription from the Svelte UI via IPC.

6. **F6: Inter-Process Communication (IPC):**

1. Establish communication between the Tauri Rust core (or its JS API in the main process) and the Svelte frontend (WebView) to trigger actions (e.g., hotkey pressed -> tell Svelte to start, Svelte wants to copy -> tell Tauri to use native clipboard if preferred).

7. **F7: Application Packaging & Installation:**

1. Generate a Windows installer (.msi or .exe).

5. **5. Technical Requirements & Specifications:**

1. **Framework:** Tauri.

2. **Frontend:** Existing Svelte/SvelteKit application.

3. **Backend Interaction:** The embedded Svelte UI will handle direct WebSocket communication with the Python backend. The Tauri shell facilitates the environment for the Svelte UI.

4. **Key Tauri APIs:** globalShortcut, SystemTray, Window (for setAlwaysOnTop, hide, show, etc.), IPC (emit, listen, invoke).

6. **6. Acceptance Criteria:**

1. **AC1:** The Svelte UI is correctly displayed and functional within the Tauri application window.

2. **AC2:** The configured global hotkey successfully starts and stops the transcription process in the Svelte UI.

3. **AC3:** The application can run in the background, and the system tray icon is present with a functional context menu.

4. **AC4:** The always-on-top/overlay mode can be activated and displays transcriptions correctly while staying on top of other windows.

5. **AC5:** Clipboard functionality via tray menu/hotkey works as expected.

6. **AC6:** Application can be packaged into a working Windows installer.

7. **7. Dependencies:**

1. Completed Svelte UI module.

2. A running Backend Transcription Service.

3. Tauri development prerequisites (Rust, Node.js, etc.) installed on the build machine.

8. **8. Out of Scope (Initially):**

1. Auto-updates for the Tauri application.

2. Complex custom window shapes for the overlay.

3. Deep OS integration beyond specified features.

## ## PRD 4: Android Shell Module (Capacitor Integration)

1. **1. Project Module:** Android Mobile Shell (Capacitor)

2. **2. Introduction & Goals:**

1. **Goal:** To package the Svelte-based UI into a native Android application using Capacitor. This shell will enable background audio capture via a Foreground Service, integrate with Nova Launcher gestures for activation, provide an optional overlay UI for viewing transcriptions, and facilitate clipboard access.

2. **Purpose:** To provide a seamless and integrated live transcription experience on Android devices, allowing users to capture voice while interacting with other apps.

3. **3. User Stories:**

1. As an Android user, I want to use a Nova Launcher gesture to quickly start or stop live transcription.

2. As an Android user, I want the transcription to continue reliably even if I switch to another app or the screen turns off.

3. As an Android user, I want an option to see the live transcription in a small overlay window while I use other apps.

4. As an Android user, I want to easily copy the transcribed text to my clipboard, possibly automatically upon completion or via an overlay button/notification action.

4. **4. Key Features & Functionality:**

1. **F1: Svelte UI Embedding:**

1. Successfully embed and display the Svelte/SvelteKit web UI within a Capacitor-managed WebView.

2. **F2: Foreground Service for Background Operation:**

1. Implement a native Android Foreground Service that:

1. Manages microphone access.

2. Maintains the WebSocket connection to the backend.

3. Streams audio and receives transcriptions even when the main app UI is not visible.

4. Displays a persistent notification indicating the service is active, with actions (e.g., Stop, Copy).

2. Use Capacitor to bridge communication between the Svelte UI and this native service (e.g., UI tells service to start/stop).

3. **F3: Nova Launcher Gesture Integration (Intent Handling):**

1. Configure the Android app to respond to being launched by a Nova Launcher gesture.

2. Implement logic (potentially via native intent filters and Capacitor bridges) to start/stop/toggle transcription based on app launch or specific intents (if Nova can send them).

4. **F4: Overlay UI (Display Over Other Apps):**

1. Request "Display over other apps" permission (SYSTEM_ALERT_WINDOW).

2. Implement a native Android overlay (e.g., a floating window or a simple view) to display live transcriptions. This overlay could be controlled by the Foreground Service or the main UI.

3. Alternatively, a custom WebView could be used for the overlay if it needs to render complex Svelte components, but this adds complexity.

5. **F5: Clipboard Integration:**

1. Provide functionality (e.g., button in overlay, notification action, or automatic on completion) to copy transcription text to the Android clipboard.

2. Use Capacitor's Clipboard plugin or custom native code.

6. **F6: Native Bridge (Capacitor Plugins):**

1. Develop custom Capacitor plugins or use existing ones to facilitate communication between the Svelte JavaScript code and native Android (Java/Kotlin) features for the foreground service, overlay, clipboard, and intent handling.

7. **F7: Application Packaging:**

1. Generate an Android App Bundle (.aab) or APK for installation.

8. **F8: Permissions Handling:**

1. Correctly request and handle necessary Android permissions (Microphone, Display over other apps, Foreground Service permissions for newer Android versions).

5. **5. Technical Requirements & Specifications:**

1. **Framework:** Capacitor.

2. **Frontend:** Existing Svelte/SvelteKit application.

3. **Native Development:** Java/Kotlin for Android Foreground Service, Overlay UI, and specific intent handlers/plugin logic.

4. **Key Capacitor APIs/Plugins:** App API, Clipboard plugin, custom plugin development.

5. **Key Android APIs:** ForegroundService, AudioRecord, WindowManager (for overlays), ClipboardManager, Intent handling.

6. **6. Acceptance Criteria:**

1. **AC1:** The Svelte UI is correctly displayed and functional within the Capacitor Android app.

2. **AC2:** Launching the app via a Nova Launcher gesture successfully starts/stops transcription as configured.

3. **AC3:** The Foreground Service starts correctly, maintains microphone access and WebSocket connection in the background, and shows a persistent notification.

4. **AC4:** (If implemented) The overlay UI can be activated, displays live transcriptions, and stays on top of other apps.

5. **AC5:** Clipboard functionality (manual or automatic) works reliably.

6. **AC6:** Necessary Android permissions are requested and handled correctly.

7. **AC7:** Application can be built into an installable Android package.

7. **7. Dependencies:**

1. Completed Svelte UI module.

2. A running Backend Transcription Service (accessible over the local network).

3. Capacitor development prerequisites (Node.js, Android Studio, SDKs) installed on the build machine.

8. **8. Out of Scope (Initially):**

1. Auto-updates for the Capacitor application.

2. Complex custom overlay shapes or advanced styling.

3. Deep Android OS integration beyond specified features.

## ## Additional Feature: Save Captured Audio

That's an excellent and very practical feature to consider! Saving local copies of captured audio is indeed valuable for post-processing, archival, or running analyses with different models/tools, especially for diarization with multiple speakers. This feature primarily impacts the client-side (Svelte UI and the Tauri/Capacitor shells) as it's generally more user-friendly for the saved files to be directly accessible on the device where the capture is happening.

Here's how we can integrate this "Save Local Audio" feature into the PRDs:

### Updates to PRD 2: Core UI Module (Svelte/SvelteKit)

1. **4. Key Features & Functionality (Additions):**

1. **F8: Local Audio Recording & Buffering (Optional):**

1. Provide a user setting (e.g., a toggle switch in the UI) to enable/disable local saving of captured audio.

2. When enabled, and transcription is active, simultaneously buffer the raw audio chunks (e.g., Float32Arrays from Web Audio API) that are being sent to the backend.

2. **F9: Audio File Preparation & Save Trigger:**

1. Upon stopping a transcription session (or via a manual "Save Audio" button if implemented), if local saving was enabled:

1. Consolidate the buffered audio chunks.

2. Convert the buffered PCM audio data into a standard audio file format (e.g., WAV format) directly in JavaScript. This involves creating the WAV header and appending the audio data.

3. Once the audio data (e.g., as a Blob or ArrayBuffer) is prepared, trigger a function (via IPC/bridge) in the parent shell (Tauri/Capacitor) to save this data to a local file, providing the audio data and a suggested filename (e.g., including a timestamp).

2. **5. Technical Requirements & Specifications (Additions):**

1. **Audio Buffering:** Manage an array or similar structure to hold audio chunks during a session.

2. **WAV Encoding:** Implement or use a lightweight JavaScript library for WAV encoding from raw PCM data.

3. **IPC for Saving:** Define a clear JavaScript interface to call the save function in Tauri/Capacitor, passing the prepared audio data.

3. **6. Acceptance Criteria (Additions):**

1. **AC8:** When "Save local audio" is enabled, audio chunks are buffered during transcription.

2. **AC9:** Upon stopping transcription (with saving enabled), the UI prepares a WAV audio blob/arraybuffer.

3. **AC10:** The UI successfully calls the designated shell function to initiate saving the prepared audio data.

4. **Documentation (for WAV encoding in JS):**

1. While many libraries exist, you can also find articles on manually creating WAV headers in JavaScript. Search for "javascript pcm to wav" or "create wav file javascript." Example conceptual logic: [Stack Overflow: Record and Save Audio in JavaScript (WAV format)](https://stackoverflow.com/questions/16369647/how-to-record-and-save-audio-in-javascript-wav-format) (look for WAV header creation parts).

### Updates to PRD 3: Windows Shell Module (Tauri Integration)

1. **4. Key Features & Functionality (Additions):**

1. **F8: Save Captured Audio File:**

1. Implement an IPC listener (e.g., Tauri invoke handler) that the Svelte UI can call to request saving an audio file.

2. This handler should receive the audio data (e.g., as a base64 string or Uint8Array if passed from JS) and a suggested filename.

3. Optionally, present a native "Save File" dialog to the user, pre-filled with the suggested filename (e.g., transcription_YYYYMMDD_HHMMSS.wav).

4. Use Tauri's file system APIs to write the received audio data to the selected file path on the Windows filesystem.

2. **5. Technical Requirements & Specifications (Additions):**

1. **Key Tauri APIs:**

1. IPC: #[tauri::command] for the Rust backend function callable from JS, or JS-based event handling.

2. File System: tauri::api::fs (e.g., writeBinaryFile) for writing the audio file.

3. Dialogs: tauri::api::dialog::save_file to let the user choose the save location and name.

2. **Data Handling:** Process binary data received via IPC correctly.

3. **6. Acceptance Criteria (Additions):**

1. **AC7:** When the Svelte UI triggers the audio save function, Tauri presents a "Save File" dialog (if implemented).

2. **AC8:** The captured audio is successfully saved as a valid WAV file at the chosen location on Windows.

3. **AC9:** File saving handles potential errors gracefully (e.g., user cancels dialog, write permission denied).

4. **Documentation:**

1. Tauri fs API: [Tauri fs (File System) API](https://tauri.app/v1/api/js/modules/fs/)

2. Tauri Dialog API: [Tauri dialog API](https://tauri.app/v1/api/js/modules/dialog/)

3. Tauri invoke (for JS to Rust communication): [Tauri Commands (Invoke)](https://tauri.app/v1/guides/features/command/)

### Updates to PRD 4: Android Shell Module (Capacitor Integration)

1. **4. Key Features & Functionality (Additions):**

1. **F9: Save Captured Audio File:**

1. Implement a Capacitor plugin method callable from the Svelte UI to save the captured audio.

2. This method should accept the audio data (e.g., as a base64 string) and a suggested filename.

3. The native Android (Java/Kotlin) part of the plugin will:

1. Decode the audio data.

2. Save the data as a WAV file to either app-specific storage or shared storage (e.g., Music or Recordings directory, respecting scoped storage guidelines for Android 10+).

3. Handle necessary file I/O permissions.

2. **5. Technical Requirements & Specifications (Additions):**

1. **Capacitor Plugin:** Develop a custom plugin if the existing Filesystem plugin doesn't fully meet the needs for saving in specific directories or handling binary blobs easily.

2. **Key Capacitor APIs/Plugins:**

1. @capacitor/filesystem plugin: ([Capacitor Filesystem API](https://capacitorjs.com/docs/apis/filesystem)) - writeFile method.

2. Custom plugin development if more control is needed.

3. **Key Android APIs:**

1. File, FileOutputStream for writing data.

2. MediaStore API for saving to shared collections.

3. Android Permissions: WRITE_EXTERNAL_STORAGE (for older Android versions or specific shared storage access, though scoped storage is preferred) and runtime permission requests.

4. **Data Handling:** Convert base64 encoded audio data from JavaScript back to binary in the native Java/Kotlin code.

3. **6. Acceptance Criteria (Additions):**

1. **AC8:** When the Svelte UI triggers the audio save function, the native Android module receives the audio data.

2. **AC9:** The captured audio is successfully saved as a valid WAV file in an appropriate location on the Android device (e.g., app's external files directory, or a shared media directory).

3. **AC10:** File saving handles permissions and potential errors gracefully.

4. **Documentation:**

1. Capacitor Filesystem Plugin: (Link provided above)

2. Android Storage Overview: [Android Storage Use Cases and Best Practices](https://developer.android.com/training/data-storage)

3. Android MediaStore: [Android MediaStore API](https://developer.android.com/reference/android/provider/MediaStore)

### Additional "Did I Miss Anything?" for this new feature:

1. **File Naming Convention:** Ensure consistency (e.g., timestamp_transcription.wav). This can be generated in the Svelte layer and passed to the shells.

2. **User Indication of Saving:** Clear visual feedback in the UI when audio recording is active and being saved (e.g., a persistent "REC" indicator, or a status message).

3. **Storage Management (Future Consideration):** While initially out of scope for automatic management, think about where files are saved so users can find them. For Android, app-specific directories are sandboxed; shared storage makes files more accessible but requires careful handling of permissions and media scanning.

4. **Audio Format:** Sticking to WAV client-side is a good starting point. If other formats are desired later (e.g., Opus for smaller files), this would require adding client-side encoding libraries (possibly WASM-based for performance).

5. **Error during Save:** What happens if the file save operation fails (disk full, permissions denied after granting, etc.)? The shell should report this back to the UI if possible.

6. **Configuration for Saving:**

1. Default save location (especially for Windows where a dialog can be optional).

2. Toggle to enable/disable saving.

This feature adds a valuable capability to your tool. Handling it client-side via the shells seems like the most user-centric approach for file access and management.

## V2 And Beyond

You're thinking ahead strategically! Those V2 and Next Gen features would significantly enhance the capabilities of your live transcription tool, building upon the solid foundation you're planning. Yes, we can definitely incorporate these into the PRD structure. V2 features can be integrated more directly into the existing module PRDs, while "Next Gen" features will be noted as future considerations, as they often involve more substantial R&D or architectural shifts.

Here are the updated PRDs, including your V2 feature ideas and noting the Next Gen possibilities:

## PRD 1: Backend Transcription Service Module (Updated for V2)

1. **1. Project Module:** Backend Transcription Service

2. **2. Introduction & Goals:**

1. **Goal:** To create a high-performance, local, real-time and batch-processing speech-to-text (STT) service. This service will run on WSL 2, leverage an NVIDIA RTX 3090 GPU via CUDA-optimized packages, and use RealtimeSTT (with faster-whisper) as its primary engine for live transcription. It will also support processing of uploaded audio files, offer selectable STT models, and provide optional speaker diarization. It will be accessible to client applications via a WebSocket interface (and potentially an HTTP endpoint for file uploads).

2. **Purpose:** To provide the core, fast, and accurate transcription engine for the Svelte-based UI running in Tauri (Windows) and Capacitor (Android) shells, supporting both live input and processing of saved audio files.

3. **3. User Stories (Illustrative, including V2):**

1. As a client application, I want to stream raw audio data to the backend service so that I can receive real-time text transcriptions.

2. As a client application, I want to receive both interim and final transcription results to display to the user.

3. **V2:** As a client application, I want to upload a previously saved audio file to the backend for transcription.

4. **V2:** As a client application, I want to be able to select from a list of available STT models/configurations on the backend before starting a transcription (live or file-based).

5. **V2:** As a client application, I want to request speaker diarization for an uploaded audio file.

6. **V2:** As a client application, I want to receive a transcript that differentiates speakers if diarization was performed.

4. **4. Key Features & Functionality:**

1. **F1: STT Engine Initialization:** (As before)

2. **F2: WebSocket Server:** (As before, may need new message types for V2 commands)

3. **F3: Live Audio Data Ingestion & Processing:** (As before - streaming audio chunks to RealtimeSTT)

4. **F4: Transcription Data Streaming (Live):** (As before - interim/final results for live audio)

5. **F5: Error Handling & Logging:** (As before)

6. **F6: Configuration:** (As before - model, port)

7. **V2-F1: Uploaded Audio File Processing:**

1. Implement a mechanism (e.g., new WebSocket message type with binary stream, or a dedicated HTTP endpoint for robust file upload) to receive complete audio files from clients.

2. Process these files using faster-whisper directly (which is excellent for file-based transcription) or adapt RealtimeSTT if suitable for batch input.

3. Return the full transcription once processing is complete.

8. **V2-F2: Selectable Processing Models:**

1. Maintain a list of available STT models/configurations (e.g., different Whisper sizes like "base", "small", "medium"; potentially different engines if added later like WhisperX for its advanced features).

2. Expose this list to clients (e.g., on WebSocket connection or via a specific request).

3. Accept a "model_choice" parameter from the client for both live and file-based transcription requests and use the selected model.

9. **V2-F3: Optional Speaker Diarization:**

1. Integrate a speaker diarization capability (e.g., using pyannote.audio or leveraging WhisperX's diarization pipeline, which uses pyannote). This is typically applied to full audio files rather than live streams due to context requirements.

2. Accept a "diarization_enabled" flag from the client for file processing requests.

3. If enabled, perform diarization and segment the transcript by speaker.

10. **V2-F4: Diarized Transcript Output:**

1. When diarization is performed, structure the output to clearly associate text segments with speaker IDs and timestamps.

2. *Example JSON for diarized output:* \
{"type": "diarized_transcript", "segments": [{"speaker": "SPEAKER_01", "start_time": 0.5, "end_time": 2.3, "text": "..."}, {"speaker": "SPEAKER_00", ...}]} \


5. **5. Technical Requirements & Specifications:**

1. (As before, with additions for V2)

2. **File Upload Handling:** If using HTTP, a library like aiohttp.web for the server. If WebSockets, careful handling of binary file streaming and reassembly or chunk-by-chunk processing.

3. **Diarization Libraries:** pyannote.audio (requires model downloads and setup), or WhisperX.

4. **WebSocket Protocol (Updates for V2):**

1. **New Client to Server message types:** \
{"type": "get_available_models"} \
{"type": "process_file_request", "filename": "...", "model_choice": "...", "diarization_enabled": true/false} \
(to be followed by binary audio data if on WebSocket, or reference a file ID if uploaded via HTTP).

2. **New Server to Client message types:** \
{"type": "available_models", "models": ["base", "small", "medium_diarize_pipeline"]} \
{"type": "file_transcript", "text": "...", "processing_details": {...}} \
{"type": "diarized_transcript", "segments": [...]} \
(as defined in V2-F4)

6. **6. Acceptance Criteria:**

1. (As before, with additions for V2)

2. **V2-AC1:** Backend can receive and save an uploaded audio file.

3. **V2-AC2:** Backend correctly processes the uploaded audio file using the client-specified model.

4. **V2-AC3:** Backend returns a list of available models/processing configurations upon request.

5. **V2-AC4:** If diarization is requested, the backend performs it and returns a transcript with speaker labels and timestamps.

6. **V2-AC5:** Transcription results for uploaded files are accurate based on the chosen model.

7. **7. Dependencies:**

1. (As before, plus diarization library dependencies like pyannote.audio and its models).

8. **8. Out of Scope (Initially):** (As before)

9. **9. Future Considerations (Next Gen):**

1. **NG-F1: User Voice Profile Training:** Implement mechanisms to collect user voice samples and train/fine-tune speaker recognition models or adapt STT for personalized transcription.

2. **NG-F2: User Identification in Diarized Transcripts:** Integrate speaker ID model with voice profiles to identify the primary user in multi-speaker scenarios.

3. **NG-F3: Secure storage and management of voice profiles.**

## PRD 2: Core UI Module (Svelte/SvelteKit) (Updated for V2)

1. **1. Project Module:** Core User Interface (Svelte/SvelteKit)

2. **2. Introduction & Goals:**

1. **Goal:** To develop a responsive and real-time web-based user interface using Svelte/SvelteKit. This UI will capture microphone audio for live transcription, allow uploading of saved audio files for batch processing, manage model selection and diarization options, display incoming transcriptions (including diarized ones), and provide basic user controls.

2. **Purpose:** (As before)

3. **3. User Stories (Including V2):**

1. (As before)

2. **V2:** As a user, I want to select a previously saved local audio file from my device and send it to the backend for transcription.

3. **V2:** As a user, I want to choose which STT model the backend should use for transcription (for both live and file-based).

4. **V2:** As a user, I want to enable or disable speaker diarization when I submit a saved audio file for processing.

5. **V2:** As a user, I want to clearly see which speaker said what when viewing a diarized transcript.

4. **4. Key Features & Functionality:**

1. **F1-F7:** (As before - Live transcription UI, WebSocket, Clipboard, Controllability Hooks, Error Display)

2. **F8: Local Audio Recording & Buffering (Optional):** (As defined in previous update)

3. **F9: Audio File Preparation & Save Trigger:** (As defined in previous update)

4. **V2-F1: Process Saved Audio File UI:**

1. Provide a UI element (e.g., "Process File" button) that triggers a file selection dialog (using the shell's capabilities).

2. Allow users to select a local audio file (e.g., WAV).

3. Once a file is selected, send it to the backend (potentially chunked over WebSockets, or as a full upload if an HTTP endpoint is used by the backend), along with selected model and diarization options.

4. Display the transcription result for the processed file.

5. **V2-F2: Model Selection Dropdown/UI:**

1. On app load or on command, request the list of available processing models/configurations from the backend via WebSocket.

2. Display these options in a dropdown or other selection UI.

3. Store the user's selection and send it to the backend with any transcription request (live or file).

6. **V2-F3: Diarization Option Toggle/UI:**

1. Provide a checkbox or toggle (likely visible when in "process file" mode) to enable/disable speaker diarization.

2. Send this boolean choice to the backend with the file processing request.

7. **V2-F4: Display Diarized Transcripts:**

1. If the backend returns a diarized transcript (e.g., with speaker labels and segments), parse this data.

2. Display the transcript in a way that clearly differentiates speakers (e.g., "Speaker A: ...", "Speaker B: ...", different colors, or a more structured table/timeline).

5. **5. Technical Requirements & Specifications:**

1. (As before)

2. **File Input Handling:** Use <input type="file"> styled appropriately, or trigger native file dialogs via shell IPC. Read file content using JavaScript FileReader API if sending via WebSocket.

3. **UI for Model/Diarization Selection:** Standard HTML select/checkbox elements styled within Svelte.

4. **Parsing Diarized JSON:** Logic to handle the structured JSON for diarized transcripts.

6. **6. Acceptance Criteria:**

1. (As before)

2. **V2-AC1:** User can select a local audio file through the UI, and it's sent to the backend along with chosen model/diarization settings.

3. **V2-AC2:** UI correctly fetches and displays available backend models in a selection element.

4. **V2-AC3:** UI allows toggling diarization, and this choice is sent with file processing requests.

5. **V2-AC4:** Diarized transcripts received from the backend are displayed clearly, differentiating speakers.

7. **7. Dependencies:** (As before)

8. **8. Out of Scope (Initially):** (As before)

9. **9. Future Considerations (Next Gen):**

1. **NG-F1: UI for Voice Profile Management:** Interface for users to record audio samples for voice training.

2. **NG-F2: UI for displaying user-identified speakers in diarized transcripts.**

## PRD 3: Windows Shell Module (Tauri Integration) (Updated for V2)

1. **1. Project Module:** Windows Desktop Shell (Tauri)

2. **2. Introduction & Goals:** (Largely the same, but facilitates new UI features)

3. **4. Key Features & Functionality:**

1. **F1-F7:** (As before - Svelte embedding, global hotkeys, background/tray, overlay, shell-level clipboard, IPC, packaging)

2. **F8: Save Captured Audio File:** (As defined in previous update)

3. **V2-F1: Native File Picker for Upload:**

1. Expose an IPC function that the Svelte UI can call to open a native Windows "Open File" dialog.

2. This dialog should ideally be filterable for common audio file types (e.g., WAV, MP3).

3. Return the path(s) of the selected file(s) back to the Svelte UI, which will then handle reading and sending the file content.

4. **5. Technical Requirements & Specifications:**

1. (As before)

2. **Key Tauri APIs (addition for V2):** tauri::api::dialog::open_file (or its equivalent in the JS API dialog.open()).

5. **6. Acceptance Criteria:**

1. (As before)

2. **V2-AC1:** Svelte UI can trigger a native Windows "Open File" dialog via Tauri IPC.

3. **V2-AC2:** The selected file path is correctly returned to the Svelte UI.

6. **7. Dependencies:** (As before)

7. **8. Out of Scope (Initially):** (As before)

8. **9. Future Considerations (Next Gen):**

1. **NG-F1:** If voice profiles are stored locally (less likely for the training part but possible for a pre-trained user model), Tauri might manage access to these secure local files.

## PRD 4: Android Shell Module (Capacitor Integration) (Updated for V2)

1. **1. Project Module:** Android Mobile Shell (Capacitor)

2. **2. Introduction & Goals:** (Largely the same, but facilitates new UI features)

3. **4. Key Features & Functionality:**

1. **F1-F8:** (As before - Svelte embedding, Foreground Service, Nova gesture integration, Overlay, Clipboard, Native Bridge, Packaging, Permissions)

2. **F9: Save Captured Audio File:** (As defined in previous update)

3. **V2-F1: Native File Picker for Upload:**

1. Implement a Capacitor plugin method (or use an existing plugin like @capacitor/file-picker) callable from Svelte to open the native Android file picker.

2. Allow filtering for audio file types.

3. Return the URI or content of the selected file(s) back to the Svelte UI for processing and sending to the backend.

4. **5. Technical Requirements & Specifications:**

1. (As before)

2. **Key Capacitor APIs/Plugins (addition for V2):** @capacitor/file-picker or similar, or custom native intent for ACTION_GET_CONTENT.

5. **6. Acceptance Criteria:**

1. (As before)

2. **V2-AC1:** Svelte UI can trigger the native Android file picker via Capacitor.

3. **V2-AC2:** The selected file's data or URI is correctly returned to the Svelte UI.

6. **7. Dependencies:** (As before)

7. **8. Out of Scope (Initially):** (As before)

8. **9. Future Considerations (Next Gen):**

1. **NG-F1:** Similar to Tauri, if any aspect of voice profile management or storage happens locally on the device, Capacitor would manage secure file access.

These updated PRDs now reflect a more feature-rich V1 and V2, and acknowledge the advanced "Next Gen" capabilities. The diarization and model selection in V2 significantly enhance the backend's role and complexity but also its utility. The "send stored audio" feature makes the tool much more versatile.
