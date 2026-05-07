# 🎙️ Real-Time Voice AI - Speech-to-Speech Conversation

## What this does
This notebook implements a natural conversational voice AI assistant that works like a real phone call:
1.  Record your voice using microphone
2.  AI automatically detects your emotion
3.  Generates natural conversational response
4.  Replies back with matching emotional voice
5.  Maintains conversation history across turns

## ✨ Features
- Real microphone recording with browser interface
- Groq ultra-fast LLM inference (Llama 3.1 8B)
- Advanced speech recognition with hallucination detection
- Emotion aware responses
- Conversation history memory (last 10 turns)
- Automatic audio playback
- Full performance timing breakdown
- T4 GPU optimized

## 🛠️ Steps to Run

### 1. Prerequisites
- Python 3.10+
- **T4 GPU recommended** (works on CPU but much slower)
- ffmpeg installed
- Groq API key (free from console.groq.com)

### 2. Install Dependencies
```bash
# System packages
sudo apt-get update && sudo apt-get install -y ffmpeg espeak-ng libsndfile1

# Python packages
pip install faster-whisper "kokoro>=0.9.4" soundfile gradio transformers accelerate torch groq
```

### 3. Configure API Key
Edit the notebook cell and add your Groq API key:
```python
GROQ_API_KEY = 'your_api_key_here'
```
Get free key at: https://console.groq.com

### 4. Run the Notebook
1.  Open `voice_ai_normal_audio.ipynb`
2.  **Run ALL cells from top to bottom**
3.  Wait for all models to load completely
4.  Final cell will launch Gradio web interface

### 5. How to Use
✅ **Workflow:**
1.  Press **Record** button
2.  Speak your question / message clearly
3.  Press **Stop** button when finished
4.  Click **▶ Send to AI** button
5.  Wait for AI response - it will play automatically
6.  Repeat to continue conversation

## 📋 Processing Pipeline
```
🎤 Microphone Recording
    ↓
✅ Audio Validation (length / volume check)
    ↓
📝 Speech Transcription (Whisper Small)
    ↓
🎭 Emotion Detection
    ↓
🤖 Groq LLM Response Generation
    ↓
🔊 Text to Speech with emotion matching
    ↓
🎧 Automatic Audio Playback
```

## ⚡ Performance
Designed for low latency conversation:
- **STT**: ~300-600ms
- **LLM**: ~200-700ms (Groq ultra fast)
- **TTS**: ~400-900ms
- **Total**: ~1-2.5 seconds per interaction

## 💡 Tips
- Speak clearly and at normal volume
- Wait for "✅ AI spoke!" before recording again
- AI remembers last 10 messages for context
- If transcription fails try speaking a little slower
