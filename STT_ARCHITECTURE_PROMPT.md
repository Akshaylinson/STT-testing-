# Speech-to-Text (STT) Architecture Framework

## 🎯 Core Concept

A **hybrid Speech-to-Text system** that combines Browser-native STT (for real-time microphone input) with Groq Whisper API (for accurate file transcription), supporting 90+ languages with native script output.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (HTML/JS)                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────┐         ┌──────────────────┐          │
│  │  🎤 Microphone   │         │  📁 File Upload  │          │
│  │  (Browser STT)   │         │  (Groq Whisper)  │          │
│  └────────┬─────────┘         └────────┬─────────┘          │
│           │                            │                     │
│           │ Real-time                  │ HTTP POST          │
│           │ Web Speech API             │ FormData           │
│           │                            │                     │
│           ▼                            ▼                     │
│  ┌─────────────────────────────────────────────┐            │
│  │      Language Selection (Manual)            │            │
│  │  - 90+ languages with search                │            │
│  │  - Native script support                    │            │
│  └─────────────────────────────────────────────┘            │
│                                                               │
└───────────────────────┬───────────────────────────────────────┘
                        │
                        │ API Calls
                        │
┌───────────────────────▼───────────────────────────────────────┐
│                    BACKEND (Flask + CORS)                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  POST /transcribe                                            │
│  ├─ Receives: audio file + language code                     │
│  ├─ Saves to: uploads/{uuid}.{ext}                           │
│  ├─ Calls: Groq Whisper API                                  │
│  │   └─ Model: whisper-large-v3-turbo                        │
│  │   └─ Language: user-selected (e.g., 'ml', 'hi', 'ta')    │
│  │   └─ Prompt: Native script hint for accuracy             │
│  └─ Returns: {text, language, elapsed, model}                │
│                                                               │
│  Environment:                                                 │
│  └─ GROQ_API_KEY from .env                                   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Components

### 1. **Dual Input Methods**

#### A. Microphone (Browser STT)
- **Technology**: Web Speech API (SpeechRecognition)
- **Pros**: Real-time, no API cost, instant feedback
- **Cons**: Requires pre-selected language, browser-dependent
- **Use Case**: Live conversations, quick notes
- **Implementation**:
  ```javascript
  const recognition = new SpeechRecognition();
  recognition.lang = 'ml-IN'; // User-selected
  recognition.continuous = true;
  recognition.interimResults = true;
  recognition.start();
  ```

#### B. File Upload (Groq Whisper API)
- **Technology**: Groq Whisper Large v3 Turbo
- **Pros**: High accuracy, supports all formats, native script output
- **Cons**: API rate limits (14,400/day free)
- **Use Case**: Pre-recorded audio, podcasts, interviews
- **Implementation**:
  ```python
  client.audio.transcriptions.create(
      file=audio_file,
      model="whisper-large-v3-turbo",
      language="ml",  # User-selected
      prompt="സാങ്കേതികവിദ്യ",  # Native script hint
      temperature=0.0
  )
  ```

---

### 2. **Language Handling**

#### Manual Selection (Current Implementation)
- **Why**: Browser STT requires language before starting
- **UX**: Searchable dropdown with 90+ languages
- **Mapping**: Browser codes (ml-IN) → Groq codes (ml)
- **Native Scripts**: Malayalam (മലയാളം), Hindi (हिन्दी), Tamil (தமிழ்), etc.

#### Language Code Mapping
```javascript
const GROQ_LANG_MAP = {
  'ml-IN': 'ml',  // Malayalam
  'hi-IN': 'hi',  // Hindi
  'ta-IN': 'ta',  // Tamil
  'te-IN': 'te',  // Telugu
  'en-IN': 'en',  // English
  // ... 85+ more
};
```

---

### 3. **Native Script Prompts**

To ensure Groq outputs in native scripts (not romanized), we use language-specific prompts:

```python
prompts = {
    "ml": "സാങ്കേതികവിദ്യ ആധുനിക ജീവിതത്തിന്റെ",
    "hi": "तकनीक आधुनिक जीवन का एक महत्वपूर्ण हिस्सा",
    "ta": "தொழில்நுட்பம் நவீன வாழ்க்கையின்",
    "te": "సాంకేతికత ఆధునిక జీవితంలో",
    "kn": "ತಂತ್ರಜ್ಞಾನ ಆಧುನಿಕ ಜೀವನದ"
}
```

This prevents output like "Sangeet vidya..." instead of "സാങ്കേതികവിദ്യ..."

---

## 📊 Data Flow

### Microphone Flow
```
User clicks mic → Selects language → Browser STT starts
→ Real-time transcription → Display with interim results
→ User stops → Final transcript shown
```

### File Upload Flow
```
User selects file → Selects language → Uploads to backend
→ Backend saves to /uploads → Calls Groq API with language + prompt
→ Groq returns native script text → Backend returns JSON
→ Frontend displays result with metadata
```

---

## 🛠️ Technical Stack

### Frontend
- **HTML5**: Structure
- **Vanilla JavaScript**: No frameworks
- **Web Speech API**: Browser STT
- **Fetch API**: Backend communication

### Backend
- **Flask**: Web framework
- **Flask-CORS**: Cross-origin support
- **Groq SDK**: Whisper API client
- **python-dotenv**: Environment variables

### API
- **Groq Whisper Large v3 Turbo**
- **Free Tier**: 14,400 requests/day
- **Speed**: 10x faster than alternatives
- **Accuracy**: Excellent for Indian languages

---

## 🎨 UI/UX Features

1. **Language Search**: Type to filter 90+ languages
2. **Tab Interface**: Switch between Mic and Upload
3. **Real-time Feedback**: Interim results for mic
4. **Result Display**: 
   - Transcription text
   - Detected language badge
   - Processing time
   - Model used
5. **Copy Button**: One-click copy to clipboard

---

## 🔐 Security & Best Practices

1. **API Key**: Stored in `.env`, never exposed to frontend
2. **CORS**: Enabled for cross-origin requests
3. **File Cleanup**: Uploaded files deleted after processing
4. **Error Handling**: Graceful fallbacks for API failures
5. **Rate Limiting**: Respect Groq's 14,400/day limit

---

## 📦 Integration Guide for Your ElevenLabs Project

### Step 1: Backend Setup
```python
# Add to your Flask app
from groq import Groq
from flask_cors import CORS

CORS(app)
GROQ_API_KEY = os.getenv("GROQ_API_KEY")

@app.route("/stt/transcribe", methods=["POST"])
def transcribe():
    file = request.files["audio"]
    language = request.form.get("language")
    
    # Save file temporarily
    save_path = f"uploads/{uuid.uuid4().hex}.wav"
    file.save(save_path)
    
    # Call Groq
    client = Groq(api_key=GROQ_API_KEY)
    result = client.audio.transcriptions.create(
        file=open(save_path, "rb"),
        model="whisper-large-v3-turbo",
        language=language,
        temperature=0.0
    )
    
    os.remove(save_path)
    return jsonify({"text": result.text})
```

### Step 2: Frontend Integration
```javascript
// Add language selector
<select id="stt-language">
  <option value="en">English</option>
  <option value="hi">Hindi</option>
  <option value="ml">Malayalam</option>
  <!-- Add more -->
</select>

// For microphone (Browser STT)
const recognition = new SpeechRecognition();
recognition.lang = document.getElementById('stt-language').value;
recognition.start();

// For file upload (Groq API)
const formData = new FormData();
formData.append('audio', audioFile);
formData.append('language', selectedLanguage);

fetch('/stt/transcribe', {
  method: 'POST',
  body: formData
}).then(res => res.json())
  .then(data => console.log(data.text));
```

### Step 3: Connect to ElevenLabs TTS
```javascript
// STT → Process → TTS Pipeline
async function voiceToVoice(audioInput, targetLanguage) {
  // 1. Transcribe with STT
  const transcript = await transcribeAudio(audioInput, targetLanguage);
  
  // 2. Process/translate if needed
  const processedText = await processText(transcript);
  
  // 3. Generate speech with ElevenLabs
  const audioOutput = await elevenLabsTTS(processedText, voiceId);
  
  return audioOutput;
}
```

---

## 🚀 Key Advantages

1. **Hybrid Approach**: Best of both worlds (real-time + accuracy)
2. **Native Scripts**: Proper Malayalam/Hindi/Tamil output
3. **90+ Languages**: Comprehensive language support
4. **Fast**: Groq is 10x faster than alternatives
5. **Free Tier**: 14,400 requests/day
6. **No Dependencies**: Minimal frontend libraries
7. **Scalable**: Easy to add more languages/features

---

## 📝 Implementation Checklist

- [ ] Install dependencies: `flask`, `groq`, `flask-cors`, `python-dotenv`
- [ ] Set `GROQ_API_KEY` in `.env`
- [ ] Create `/uploads` folder
- [ ] Add language dropdown with search
- [ ] Implement Browser STT for microphone
- [ ] Create `/transcribe` endpoint for file upload
- [ ] Add language code mapping (Browser ↔ Groq)
- [ ] Include native script prompts for Indian languages
- [ ] Test with Malayalam/Hindi/Tamil audio
- [ ] Add error handling and loading states
- [ ] Implement copy-to-clipboard functionality

---

## 🎯 Use This Prompt for Your ElevenLabs Project

**"Implement a hybrid Speech-to-Text system with:**
1. **Browser STT** for real-time microphone input (Web Speech API)
2. **Groq Whisper API** for accurate file transcription
3. **Manual language selection** with 90+ languages (searchable dropdown)
4. **Native script support** for Indian languages (Malayalam, Hindi, Tamil, Telugu, Kannada)
5. **Flask backend** with `/transcribe` endpoint
6. **Language code mapping** between browser codes (ml-IN) and Groq codes (ml)
7. **Native script prompts** to prevent romanized output
8. **CORS enabled** for cross-origin requests
9. **File cleanup** after processing
10. **Result display** with transcription, language, and processing time

**Key requirement**: User selects language first, then either speaks (Browser STT) or uploads file (Groq API). Output must be in native script (സാങ്കേതികവിദ്യ not 'Sangeet vidya')."

---

## 📚 References

- Groq API Docs: https://console.groq.com/docs
- Web Speech API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API
- Whisper Model: https://github.com/openai/whisper
- Language Codes: ISO 639-1 + BCP 47

---

**Architecture Version**: 1.0  
**Last Updated**: 2024  
**Compatibility**: Flask 3.0+, Modern Browsers (Chrome, Edge, Safari)
