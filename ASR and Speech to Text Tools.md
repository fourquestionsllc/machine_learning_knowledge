Here are some of the most popular **Automatic Speech Recognition (ASR)** and **Speech-to-Text (STT)** tools:

---

### 1. **Google Speech-to-Text API**
- **Description**: A widely used, accurate, and scalable cloud service from Google for real-time ASR and STT.
- **Use Cases**: Transcribing calls, creating subtitles, voice search, and more.
- **Language Support**: Over 120 languages.

#### Code Example

```python
from google.cloud import speech_v1p1beta1 as speech

# Initialize the client
client = speech.SpeechClient()

# Path to the audio file
audio_file_path = "path_to_audio.wav"

# Load audio file
with open(audio_file_path, "rb") as audio_file:
    content = audio_file.read()

audio = speech.RecognitionAudio(content=content)
config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=16000,
    language_code="en-US",
)

# Perform speech recognition
response = client.recognize(config=config, audio=audio)

# Print the transcription
for result in response.results:
    print("Transcript: {}".format(result.alternatives[0].transcript))
```

---

### 2. **OpenAI Whisper**
- **Description**: An open-source ASR model developed by OpenAI.
- **Use Cases**: Transcription, translation, and real-time ASR applications.
- **Language Support**: Multilingual.

#### Code Example

```python
import whisper

# Load the model
model = whisper.load_model("base")

# Transcribe the audio
result = model.transcribe("path_to_audio.wav")

# Print the transcription
print(result["text"])
```

---

### 3. **Microsoft Azure Speech-to-Text**
- **Description**: A powerful cloud-based ASR service from Microsoft.
- **Use Cases**: Real-time transcription, voice commands, and more.
- **Language Support**: Over 100 languages.

#### Code Example

```python
import azure.cognitiveservices.speech as speechsdk

# Initialize the Speech Config
speech_config = speechsdk.SpeechConfig(
    subscription="YourSubscriptionKey",
    region="YourRegion"
)

# Set the audio config (from file or microphone)
audio_config = speechsdk.audio.AudioConfig(filename="path_to_audio.wav")

# Create the Speech Recognizer
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_config)

# Perform recognition
result = speech_recognizer.recognize_once()

# Print the result
if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print("Recognized: {}".format(result.text))
else:
    print("Error: {}".format(result.reason))
```

---

### 4. **AWS Transcribe**
- **Description**: Amazon’s fully managed ASR service for transcribing audio to text.
- **Use Cases**: Call analytics, live transcription, and subtitles.
- **Language Support**: 31 languages.

#### Code Example

```python
import boto3

# Initialize the Transcribe client
transcribe = boto3.client('transcribe', region_name='us-east-1')

# Start transcription job
response = transcribe.start_transcription_job(
    TranscriptionJobName="MyTranscriptionJob",
    Media={"MediaFileUri": "s3://bucket_name/path_to_audio.mp4"},
    MediaFormat="mp4",
    LanguageCode="en-US"
)

# Print the transcription job details
print(response)
```

---

### 5. **IBM Watson Speech-to-Text**
- **Description**: IBM’s ASR tool designed for scalability and accuracy.
- **Use Cases**: Call transcription, real-time applications, and closed captions.
- **Language Support**: Over 10 languages.

#### Code Example

```python
from ibm_watson import SpeechToTextV1
from ibm_cloud_sdk_core.authenticators import IAMAuthenticator

# Authenticate the service
authenticator = IAMAuthenticator('YourAPIKey')
speech_to_text = SpeechToTextV1(authenticator=authenticator)
speech_to_text.set_service_url('YourServiceURL')

# Read audio file
with open('path_to_audio.wav', 'rb') as audio_file:
    result = speech_to_text.recognize(
        audio=audio_file,
        content_type='audio/wav'
    ).get_result()

# Print the transcription
print(result['results'][0]['alternatives'][0]['transcript'])
```

---

### 6. **AssemblyAI**
- **Description**: A developer-friendly API for real-time and batch speech-to-text applications.
- **Use Cases**: Podcast transcription, voice analytics, and video subtitling.
- **Language Support**: English.

#### Code Example

```python
import requests

# API URL and key
url = "https://api.assemblyai.com/v2/transcript"
headers = {
    "authorization": "YourAPIKey",
    "content-type": "application/json"
}

# File upload
audio_url = "https://example.com/path_to_audio.mp3"

# Send request
response = requests.post(url, headers=headers, json={"audio_url": audio_url})

# Print the transcription
print(response.json())
```

---

### Comparison Table

| Tool              | Open Source | Cloud-Based | Multilingual | Real-Time Support |
|-------------------|-------------|-------------|--------------|-------------------|
| Google STT        | ❌           | ✅           | ✅            | ✅                |
| OpenAI Whisper    | ✅           | ❌           | ✅            | ❌ (batch only)   |
| Microsoft Azure   | ❌           | ✅           | ✅            | ✅                |
| AWS Transcribe    | ❌           | ✅           | ✅            | ✅                |
| IBM Watson        | ❌           | ✅           | ✅            | ✅                |
| AssemblyAI        | ❌           | ✅           | ❌            | ✅                |

--- 

### Recommendations
- **Use Google Speech-to-Text API** for robust, highly scalable, and multilingual real-time applications.
- **Use OpenAI Whisper** for open-source and offline transcription tasks.
- **Use AWS Transcribe or Azure Speech-to-Text** if you’re already on their cloud ecosystems.
