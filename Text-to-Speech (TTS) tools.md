Here are some of the most popular Text-to-Speech (TTS) tools available now:

---

### 1. **Google Text-to-Speech (via gTTS)**

#### Features:
- Free and easy to use.
- Supports multiple languages and accents.
- Suitable for basic TTS needs.

#### Installation:
```bash
pip install gTTS
```

#### Example Code:
```python
from gtts import gTTS

# Text to convert to speech
text = "Hello, welcome to the world of text to speech!"

# Convert to speech
tts = gTTS(text=text, lang='en', slow=False)

# Save the audio file
tts.save("output.mp3")

# Play the audio file
import os
os.system("start output.mp3")
```

---

### 2. **Amazon Polly**

#### Features:
- High-quality voices, including neural TTS for lifelike speech.
- Multiple voices and languages.
- Pay-as-you-go pricing.

#### Installation:
```bash
pip install boto3
```

#### Example Code:
```python
import boto3

# Initialize the Polly client
polly = boto3.client('polly', region_name='us-west-2')

# Input text
text = "Amazon Polly converts text into lifelike speech!"

# Request speech synthesis
response = polly.synthesize_speech(
    Text=text,
    OutputFormat='mp3',
    VoiceId='Joanna'
)

# Save the audio stream to a file
with open("output.mp3", "wb") as file:
    file.write(response['AudioStream'].read())

# Play the audio file
import os
os.system("start output.mp3")
```

---

### 3. **Microsoft Azure Speech Service**

#### Features:
- High-quality, customizable voices.
- Support for neural TTS.
- Can adjust pitch, speed, and volume.

#### Installation:
```bash
pip install azure-cognitiveservices-speech
```

#### Example Code:
```python
import azure.cognitiveservices.speech as speechsdk

# Set up the Azure Speech configuration
speech_config = speechsdk.SpeechConfig(
    subscription="YourAzureSubscriptionKey",
    region="YourRegion"
)

# Create a synthesizer instance
synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)

# Text to synthesize
text = "This is Microsoft Azure's Text-to-Speech service!"

# Generate speech
synthesizer.speak_text_async(text)
```

---

### 4. **Coqui TTS**

#### Features:
- Open-source TTS solution.
- Supports multiple languages.
- Highly customizable with training support.

#### Installation:
```bash
pip install tts
```

#### Example Code:
```python
from TTS.api import TTS

# Initialize TTS with a pre-trained model
tts = TTS(model_name="tts_models/en/ljspeech/tacotron2-DDC", gpu=False)

# Text to convert to speech
text = "This is an open-source text-to-speech tool by Coqui."

# Generate and save audio
tts.tts_to_file(text=text, file_path="output.wav")
```

---

### 5. **ElevenLabs**

#### Features:
- High-quality voices with emotional inflection.
- Paid service with API access.
- Focuses on creating lifelike speech.

#### Installation:
```bash
pip install elevenlabs
```

#### Example Code:
```python
from elevenlabs import generate, play

# Set API key
import os
os.environ["ELEVENLABS_API_KEY"] = "Your_API_Key"

# Generate speech
audio = generate(
    text="Hello! This is ElevenLabs Text-to-Speech service.",
    voice="Rachel"
)

# Play the audio
play(audio)
```

---

### Summary

Each TTS tool has its strengths, depending on your use case:
- **gTTS**: Best for quick and free usage.
- **Amazon Polly** and **Azure**: Enterprise-grade quality with flexible APIs.
- **Coqui TTS**: Ideal for developers looking for an open-source option.
- **ElevenLabs**: Excellent for lifelike and emotional TTS.
