---
title: 🎙️ KANILAK Wake Word
tags:
  - project/python
  - voice-assistant
  - speech-recognition
  - picovoice
  - tts
creation_date: 2025-08-07
---

> ### 🧠 Project Overview
    A custom voice assistant built with **Python**, using **Picovoice Porcupine** for wake word detection (🪄"Candy"), and **Whisper** for speech-to-text transcription. The assistant can respond to commands like telling the time using TTS.

---

## 🧩 Module Imports

```python
import datetime
import os
import struct
import webbrowser
import pyttsx3
import speech_recognition as sr
import pvporcupine
import pyaudio
```

> ✅ **Explanation**
- 🕒 `datetime`: Gets the current time.
- 📁 `os`: Handles file operations.
- 🧱 `struct`: Converts raw audio data.
- 🌐 `webbrowser`: Can be used to open websites (potential extension).
- 🔊 `pyttsx3`: Text-to-speech engine.
- 🗣️ `speech_recognition`: Captures and transcribes voice input.
- 🎙️ `pvporcupine`: Wake word detection using Picovoice.
- 🎧 `pyaudio`: Real-time audio streaming from mic.

---

## 🔑 Configuration & Initialization

```python
PICOVOICE_ACCESS_KEY = "your_access_key_here"
WAKE_WORD_FILE_PATH = "Candy_en_windows_v3_0_0.ppn"
```

> 💡 **Details**
- 🔐 **Access Key**: Required for authenticating with Picovoice.
- 📄 **Wake Word File**: The `.ppn` model trained to detect "Candy".

---

## 🗣️ TTS Initialization

```python
try:
    engine = pyttsx3.init()
    voices = engine.getProperty('voices')
    engine.setProperty('voice', voices[0].id)
    engine.setProperty('rate', 180)
```

> 🧠 **Explanation**
- Initializes the speech engine.
- Selects default voice and sets speed to 180 WPM.

> ⚠️ If initialization fails, it sets `engine = None` and prints the error.

---

## 🧵 `speak()` Function

```python
def speak(text):
    if not engine: return
    print(f"Kanilak: {text}")
    engine.say(text)
    engine.runAndWait()
```

> 🗯️ Speaks out a given `text`.  
> 📟 Also prints it to console for visibility.

---

## ⚙️ `process_command()` Function

```python
def process_command(command):
    if 'time' in command:
        now = datetime.datetime.now().strftime("%I:%M %p")
        return f"The current time is {now}."
    else:
        return "I'm not sure how to help with that yet."
```

> 🧠 **Logic Layer**
- If user says "time", it fetches and formats the current time.
- Else, returns a default fallback.

> 📦 You can add more commands here like “open YouTube”, “weather”, etc.

---

## 🎤 `listen_and_transcribe()` Function

```python
def listen_and_transcribe():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        speak("How can I help you?")
        ...
        command = r.recognize_whisper(audio, model="base", language="english")
        ...
```

> 🔍 **Workflow**
1. Initializes recognizer.
2. Listens to mic input.
3. Uses **OpenAI Whisper** (`recognize_whisper`) for transcription.
4. Returns lowercased command string.

> ⏱️ **Timeouts**:
- `timeout=5`: Max time to wait for speaking to begin.
- `phrase_time_limit=10`: Max duration of the sentence.

> ❗ Catches errors like silence, recognition failures.

---

## 🪄 Wake Word Detection: `listen_for_wake_word()`

```python
porcupine = pvporcupine.create(
    access_key=PICOVOICE_ACCESS_KEY,
    keyword_paths=[WAKE_WORD_FILE_PATH]
)
```

> 🚨 **This activates Picovoice Porcupine**:
- Loads your `.ppn` wake word model.
- Listens in real-time for the word "Candy".

```python
audio_stream = pa.open(...)
pcm = audio_stream.read(porcupine.frame_length)
keyword_index = porcupine.process(pcm)
```

> 🎧 **How it works**
- Audio is read in small chunks (`frame_length`).
- Chunks are checked using `porcupine.process(pcm)`.
- When the wake word is heard, it returns `keyword_index >= 0`.

> 🔄 **Loop keeps running until wake word is detected.**

> 💣 On error, it prints and exits gracefully.

---

## 🏁 Main Loop

```python
if __name__ == "__main__":
    while True:
        if listen_for_wake_word():
            command = listen_and_transcribe()
            if command:
                response = process_command(command)
                speak(response)
```

> 🔂 **Continuous Execution**
- Waits for wake word ("Candy").
- Once detected, listens for a voice command.
- Responds based on logic.

---

## 💡 How It All Works Together

```mermaid
graph TD
    A[User says "Candy"] --> B[Porcupine detects wake word]
    B --> C[Microphone activated]
    C --> D[Whisper transcribes voice]
    D --> E[Command parsed]
    E --> F[Response generated]
    F --> G[TTS speaks response]
```

---

## 📁 Directory Structure

```
/your_project_folder
│
├── main.py
├── Candy_en_windows_v3_0_0.ppn
├── requirements.txt
```

> ✅ Place your `.ppn` file in the **same directory** as your script.

---

## 🔧 Requirements

> Run this in a terminal or `requirements.txt`

```
pyttsx3
SpeechRecognition
pvporcupine
pyaudio
```

> 📦 Also ensure:
- `whisper` is installed (`pip install openai-whisper`)
- `ffmpeg` is installed in your system (for Whisper to work)

---

## 🏁 Future Improvements

- 🧠 Add more natural language understanding
- 🌐 Web search support
- 🔔 Reminders and alarms
- 🤖 GPT-based responses
- 💬 Chat logs & GUI

---

> 💬 *"Kanilak ready to serve."*


AIzaSyCF7HeYL9badl3U6RDBTcTEvt5SEXok4kE