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

### Habit Tracker PWA

**📂 [View on GitHub](https://github.com/sumitgouthaman/habit-tracker)**

A PWA for tracking daily, weekly, and monthly habits with offline support and stats.

**Features:**
- **Smart Tracking**: Daily, Weekly, and Monthly habits with auto-reset.
- **Flexible Goals**: Track binary completion (Done/Not Done) or specific counts.
- **Visual Stats**: View streaks and completion progress.
- **Offline Ready**: Works without internet connection.
- **Clean UI**: Dark mode design with glassmorphism elements.

<div style="display: flex; gap: 10px; overflow-x: auto; padding: 10px 0;">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/main_page.png" alt="Habit Tracker Main Page" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/stats.png" alt="Habit Tracker Stats" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/stats_2.png" alt="Habit Tracker Stats 2" width="200">
  <img src="https://raw.githubusercontent.com/sumitgouthaman/habit-tracker/main/screenshots/settings.png" alt="Habit Tracker Settings" width="200">
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
