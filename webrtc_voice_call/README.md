# 📞 WebRTC Real-Time Voice AI Call

## What this does
This notebook implements a **live real-time two-way voice call** with AI using WebRTC technology. This is not a send/receive workflow - you have a natural continuous conversation with AI just like a real phone call:
1.  Direct browser microphone streaming
2.  Real-time voice activity detection
3.  Automatic end-of-speech detection
4.  Live AI response streaming
5.  Echo cancellation and full duplex audio
6.  Works anywhere via ngrok public tunnel

## ✨ Features
- **True full-duplex WebRTC audio**
- Automatic Voice Activity Detection (VAD)
- Smart end-of-speech detection
- Echo cancellation (AI doesn't hear itself)
- Groq ultra fast LLM inference
- Emotion aware responses
- Conversation history memory
- Beautiful modern web interface
- Real time mic level indicator
- Conversation transcript log
- TURN/STUN servers for worldwide connectivity

## 🛠️ Steps to Run

### 1. Prerequisites
- Python 3.10+
- **T4 GPU Required** for acceptable latency
- API Keys required:
  - Groq API Key (free: console.groq.com)
  - Ngrok Auth Token (free: dashboard.ngrok.com)
  - Metered.ca TURN credentials (free: metered.ca)

### 2. Install Dependencies
```bash
# System packages
sudo apt-get install -y ffmpeg espeak-ng libsndfile1 libopus-dev libvpx-dev

# Python packages
pip install faster-whisper "kokoro>=0.9.4" soundfile groq aiortc aiohttp av fastapi uvicorn pyngrok numpy
```

### 3. Configure API Keys
Edit the configuration cell and add your API keys:
```python
GROQ_API_KEY = 'your_groq_api_key'
NGROK_TOKEN  = 'your_ngrok_auth_token'
METERED_USERNAME   = 'your_metered_username'
METERED_CREDENTIAL = 'your_metered_password'
```

### 4. Run the Notebook
1.  Open `webrtc_voice_call.ipynb`
2.  **Run ALL cells in order from top to bottom**
3.  Wait for all models to load completely
4.  Last cell will print a public ngrok URL
5.  Open the ngrok URL in any browser on any device

### 5. How to Use
✅ **Call workflow:**
1.  Click the green 📞 Call button
2.  Allow microphone permission when prompted
3.  Wait for "✅ Connected — speak now" status
4.  Just speak normally — you don't need to press any buttons
5.  AI will automatically reply when you finish speaking
6.  Continue conversation naturally just like a phone call
7.  Click red 🔴 button to hang up

## 🔊 Audio Pipeline
```
🎤 Browser Microphone
    ↓
🔌 WebRTC Encrypted Stream
    ↓
⚡ Real-time VAD + Silence Detection
    ↓
⏸️ Auto stop when you finish talking
    ↓
📝 Speech Transcription (Whisper Medium)
    ↓
🎭 Emotion Detection
    ↓
🤖 Groq LLM Response
    ↓
🔊 Natural Text to Speech
    ↓
🔌 WebRTC Audio Stream back to browser
    ↓
🎧 Automatic audio playback
```

## ⚡ Performance
This is optimized for sub-second latency:
- **VAD Detection**: ~200ms
- **STT**: ~400-700ms
- **LLM**: ~200-600ms
- **TTS**: ~400-800ms
- **Total**: ~1.2-2.5 seconds from end of speech to AI reply

## 💡 Important Notes
- Works on any modern browser (Chrome, Edge, Safari, Firefox)
- Works on mobile phones too
- Requires HTTPS for microphone access (ngrok provides this automatically)
- AI automatically pauses listening while speaking to prevent echo
- All audio processing happens in real time
- No button presses required during conversation - just talk naturally
