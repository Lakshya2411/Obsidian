---
creation_date: 2025-08-06
tags:
  - project/ML
  - project/Python
  - topic/VoiceAssistant
  - state/completed
aliases: [Jarvis Project, Python Voice Assistant]
---

# 🤖 Project: Building a JARVIS-Like Voice Assistant

This project outlines the steps to create a modular voice assistant in Python. The system is broken down into independent components, making it easier to build, test, and upgrade.

**Core Architecture:**
1.  **Wake Word Engine** → Listens for "Jarvis"
2.  **Speech-to-Text (STT)** → Transcribes your command
3.  **Action Processor (NLU)** → Understands and executes the command
4.  **Text-to-Speech (TTS)** → Speaks the response

---

## ✅ Prerequisites Checklist

- [ ] Python 3.8+ installed
- [ ] Pip package manager
- [ ] Microphone connected and configured
- [ ] Run the installation command in your terminal:
  ```bash
  pip install pvporcupine pyaudio speechrecognition openai-whisper pyttsx3
  ```

> [!CAUTION] API Key Required  
> This project requires a free **Access Key** for the wake word engine.  
> Get yours from the [Picovoice Console](https://console.picovoice.ai/).

---

## ## 1. Wake Word Detection

> [!NOTE] File: `wake_word.py`

### Purpose

Listens continuously for the keyword **"Jarvis"** using the lightweight `Porcupine` engine. Once detected, it returns `True` and releases the microphone resources.

### Code

```python
# wake_word.py
import pvporcupine
import pyaudio
import struct

def listen_for_wake_word():
    access_key = "YOUR_PICOVOICE_ACCESS_KEY" # <--- IMPORTANT!

    handle = pvporcupine.create(access_key=access_key, keywords=['jarvis'])
    pa = pyaudio.PyAudio()
    
    audio_stream = pa.open(
        rate=handle.sample_rate,
        channels=1,
        format=pyaudio.paInt16,
        input=True,
        frames_per_buffer=handle.frame_length
    )
    print("Listening for 'Jarvis'...")

    while True:
        pcm = audio_stream.read(handle.frame_length)
        pcm = struct.unpack_from("h" * handle.frame_length, pcm)
        keyword_index = handle.process(pcm)

        if keyword_index >= 0:
            print("Wake word detected!")
            audio_stream.close()
            pa.terminate()
            handle.delete()
            return True
```

### Code Explanation

- `access_key`: Replace with your **actual key**.
- `pvporcupine.create()`: Initializes the wake word engine.
- `pyaudio.open()`: Opens microphone stream.
- `handle.process(pcm)`: Processes audio to detect the wake word.
- Cleans up with `close()`, `terminate()`, and `delete()`.

---

## ## 2. Speech-to-Text (STT)

> [!NOTE] File: `transcriber.py`

### Purpose

To accurately convert spoken audio into a text string for our program to understand.

### Code

```python
# transcriber.py
import speech_recognition as sr

def listen_and_transcribe():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening for command...")
        r.adjust_for_ambient_noise(source)
        audio = r.listen(source)

    try:
        command = r.recognize_whisper(audio, model="base", language="english")
        print(f"You said: {command}")
        return command.lower()
    except sr.UnknownValueError:
        print("Whisper could not understand the audio.")
        return None
    except sr.RequestError as e:
        print(f"Whisper service error; {e}")
        return None
```

### Code Explanation

- `Recognizer()` handles transcription.
- `adjust_for_ambient_noise()` improves accuracy.
- `recognize_whisper()` uses Whisper to transcribe.
- Returns lowercase string or `None` on failure.

---

## ## 3. Action Processing

> [!NOTE] File: `actions.py`

### Purpose

To determine the user's **intent** and execute the corresponding code.

### Code

```python
# actions.py
import datetime
import webbrowser
import os

def process_command(command):
    if 'time' in command:
        now = datetime.datetime.now().strftime("%I:%M %p")
        return f"The current time is {now}."

    elif 'search for' in command:
        query = command.replace('search for', '').strip()
        webbrowser.open(f"https://www.google.com/search?q={query}")
        return f"Searching for {query} on Google."

    elif 'open code' in command:
        os.startfile("C:\Users\YourUser\AppData\Local\Programs\Microsoft VS Code\Code.exe")
        return "Opening Visual Studio Code."
    
    else:
        return "I'm not sure how to help with that yet."
```

### Code Explanation

- `if 'keyword' in command`: Basic intent recognition.
- `replace()`: Extracts search query.
- `webbrowser.open()`: Opens URL.
- `os.startfile()`: Launches VS Code (Windows only).

---

## ## 4. Text-to-Speech (TTS)

> [!NOTE] File: `speaker.py`

### Purpose

To convert the assistant's response into speech.

### Code

```python
# speaker.py
import pyttsx3

engine = pyttsx3.init()
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)
engine.setProperty('rate', 180)

def speak(text):
    print(f"Jarvis: {text}")
    engine.say(text)
    engine.runAndWait()
```

### Code Explanation

- `pyttsx3.init()`: Initializes TTS engine.
- `setProperty()`: Changes voice and speed.
- `say()` and `runAndWait()`: Converts and plays audio.

---

## ## 5. Orchestration & Main Loop

> [!NOTE] File: `main.py`

### Purpose

To combine all components into a continuous operational cycle.

### Code

```python
# main.py
from wake_word import listen_for_wake_word
from transcriber import listen_and_transcribe
from actions import process_command
from speaker import speak

def main_loop():
    if listen_for_wake_word():
        speak("Yes sir?")
        command = listen_and_transcribe()
        
        if command:
            response = process_command(command)
            speak(response)

if __name__ == "__main__":
    print("Jarvis assistant started...")
    while True:
        main_loop()
```

### Code Explanation

- Imports all modules.
- `main_loop()` runs the assistant cycle.
- `while True:` keeps it always on.

> [!TIP] How to Run
>
> 1. Save all `.py` files in the same folder.
> 2. Replace the API key in `wake_word.py`.
> 3. Open terminal in that folder.
> 4. Run: `python main.py`

---

> [!TODO] Future Improvements
>
> - [ ] Add LLM support for complex commands.
> - [ ] Include smart home or music APIs.
> - [ ] Implement memory for multi-turn dialogue.
> - [ ] Upgrade TTS with ElevenLabs.