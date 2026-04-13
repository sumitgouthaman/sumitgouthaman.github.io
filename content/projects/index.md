---
title: 'Projects'
hidden: true

menu:
  main:
    weight: 2
---

## Vibe coding experiments

Quick projects built while experimenting with various tools and approaches.

### Podcast Smart Trim

**📂 [View on GitHub](https://github.com/sumitgouthaman/podcast-smart-trim)**

A CLI tool that uses AI to automatically identify and remove advertisements from podcasts. This project was built to test **Antigravity**, Google's new IDE.

**Features:**
- **AI Transcription**: Uses `openai-whisper` for accurate transcripts.
- **Smart Ad Detection**: Uses Google Gemini to analyze context and identify ad breaks.
- **Ad Removal**: Uses `ffmpeg` to remove the identified ad segments.

### BusDash: Android & Wear OS Transit Dashboard

**📂 [View on GitHub](https://github.com/sumitgouthaman/BusDash)**

A fast, glanceable transit dashboard for Android and Wear OS. Built for daily commuters who take the same bus from the same stops every day, powered by the OneBusAway API.

**Features:**
- **Real-time Arrivals**: Handles API rate limits gracefully with caching.
- **Location-aware**: Shows nearby stops automatically and floats starred stops to the top.
- **Wear OS Companion**: Syncs starred stops and settings from your phone.
- **Dark UI**: High-contrast, transit-themed dark mode throughout.

<div style="display: flex; gap: 10px; overflow-x: auto; padding: 10px 0; align-items: center;">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/BusDash/main/screenshots/screenshot_phone.png" alt="BusDash Phone" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/BusDash/main/screenshots/recording_phone.gif" alt="BusDash Phone Demo" width="200">
  <div style="display: flex; flex-direction: column; gap: 10px; align-items: center; justify-content: center;">
    <img src="https://raw.githubusercontent.com/sumitgouthaman/BusDash/main/screenshots/screenshot_watch.png" alt="BusDash Watch" width="150" style="object-fit: contain;">
    <img src="https://raw.githubusercontent.com/sumitgouthaman/BusDash/main/screenshots/recording_watch.gif" alt="BusDash Watch Demo" width="150" style="object-fit: contain;">
  </div>
</div>

### Habit Tracker: Web & Wear OS

**🚀 [Try it live here!](https://habit-tracker.sumitgouthaman.com)** | **[View on GitHub](https://github.com/sumitgouthaman/habit-tracker)**

A seamless cross-device habit tracker with a web PWA and a dedicated Wear OS companion app.

**Features:**
- **Smart Tracking**: Daily, Weekly, and Monthly habits with auto-reset.
- **Flexible Goals**: Track binary completion (Done/Not Done) or specific counts.
- **Wear OS Companion**: Check off your habits directly from your wrist (built with Jetpack Compose).
- **Visual Stats**: View streaks and completion progress.
- **Offline Ready**: Works without an internet connection.
- **Clean UI**: Dark mode design with glassmorphism elements.

<div style="display: flex; gap: 10px; overflow-x: auto; padding: 10px 0;">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/main_page.png" alt="Habit Tracker Main Page" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/stats.png" alt="Habit Tracker Stats" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/stats_3.png" alt="Detailed Stats" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/settings.png" alt="Habit Tracker Settings" width="200">
</div>

**Wear OS App:**
<div style="display: flex; gap: 10px; overflow-x: auto; padding: 10px 0;">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/android/screenshots/wearos_screenshot.png" alt="Habit Tracker Wear OS" width="150">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/android/screenshots/wearos_recording.gif" alt="Habit Tracker Wear OS Demo" width="150">
</div>

### OBA macOS — Bus Arrivals in Your Menu Bar

**📂 [View on GitHub](https://github.com/sumitgouthaman/oba-macos)**

A native macOS menu bar app that shows real-time bus arrivals for your saved stops via the OneBusAway API.

<div style="text-align: center;">
<img src="https://raw.githubusercontent.com/sumitgouthaman/oba-macos/main/screenshots/popover.png" alt="OBA macOS popover" width="300">
</div>

### pdf-cropper — Per-Page PDF Cropping

**📂 [View on GitHub](https://github.com/sumitgouthaman/pdf-cropper)**

A locally-hosted web app for cropping individual pages of a PDF — each page independently. Most PDF tools apply the same crop rectangle to every page, which doesn't work well for scanned books or mixed-format PDFs. This tool was built to handle exactly that case.

Crop each PDF page independently, with a sidebar thumbnail preview. Runs locally — no cloud services.

<div style="text-align: center;">
<img src="https://raw.githubusercontent.com/sumitgouthaman/pdf-cropper/main/screenshots/recording.gif" alt="pdf-cropper demo" width="700">
</div>

### Morse Code Trainer

**🚀 [Try it live here!](https://morse-code.sumitgouthaman.com)** | **📂 [View on GitHub](https://github.com/sumitgouthaman/morse-code-trainer)**

An interactive web application for practicing morse code. Built using Gemini CLI and Q CLI as a rapid development experiment.

**Features:**
- Multiple practice modes (character ↔ morse, audio recognition)
- Learn mode for studying patterns
- Mobile-responsive with PWA support

<div style="text-align: center;">
<img src="https://raw.githubusercontent.com/sumitgouthaman/morse-code-trainer/main/_screenshots/local/mobile/combined.gif" alt="Morse Code Trainer Demo" width="300">
</div>

*Built with vanilla JavaScript and the HTML5 Audio API.*


### iPad Photo Frame

**📂 [View on GitHub](https://github.com/sumitgouthaman/ipad-photo-frame)**

Turn your unused iPad into a digital photo frame—no cloud services, logins, or remote backends required. I wanted a simple way to repurpose an old iPad without relying on third-party services, and with the ability to easily add new photos without connecting a cable. The app exposes a local web server, letting you drag and drop photos from any browser on your network. This approach was inspired by a similar feature in VLC Player for iOS.

**Features:**
- **Fully Offline**: No cloud services or accounts required.
- **Wireless Uploads**: Drag and drop photos from any browser on your network.
- **No Cables**: Add new photos without connecting to a computer.
- **Privacy First**: Your photos stay on your device.
