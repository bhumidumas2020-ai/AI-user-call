# 📞 AI Call Agent with Emotion Detection

## What this does
This notebook implements a complete end-to-end voice AI assistant that:
1.  Takes user voice audio input
2.  Transcribes speech to text with noise reduction
3.  Detects user emotion from conversation
4.  Generates appropriate empathetic AI response
5.  Converts AI response back to natural voice with matching emotion
6.  Provides full processing timing analytics

## ✨ Features
- Real-time Speech-to-Text (Faster Whisper)
- Local LLM inference (Ollama / Phi model)
- Emotion detection from conversation keywords
- Natural Text-to-Speech (Kokoro TTS)
- Interactive Gradio web UI
- Full processing performance timing
- Noise reduction on audio input
- GPU acceleration support

## 🛠️ Steps to Run

### 1. Prerequisites
- Python 3.10+
- CUDA GPU recommended (works on CPU too)
- ffmpeg installed on system

### 2. Install Dependencies
```bash
# System packages
sudo apt-get update && sudo apt-get install -y zstd ffmpeg espeak-ng

# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Python packages
pip install faster-whisper kokoro>=0.9.4 soundfile gradio ffmpeg-python requests
```

### 3. Prepare LLM Model
```bash
# Start Ollama service
ollama serve &

# Pull Phi model
ollama pull phi
```

### 4. Run the Notebook
1.  Open `voice_ai_file.ipynb`
2.  Run cells in order from top to bottom
3.  Final cell will launch Gradio web interface with public share link
4.  Upload audio file or record voice to interact with the AI

## 📋 Pipeline Workflow
```
🎤 Audio Input
    ↓
🔊 Noise Reduction + Conversion
    ↓
📝 Speech Transcription (Whisper)
    ↓
🎭 Emotion Detection
    ↓
🤖 LLM Response Generation
    ↓
🔊 Text to Speech with emotion voice
    ↓
🎧 Audio Output + Timing Report
```

## ⚡ Performance Expectations
- **STT**: ~200-400ms
- **LLM**: ~500-1500ms  
- **TTS**: ~300-800ms
- **Total**: ~1-3 seconds per interaction
