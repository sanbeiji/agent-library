# Gemini Text-to-Speech (TTS) Skill

## Specification
The Gemini TTS skill provides advanced text-to-speech capabilities, leveraging Gemini's natural language understanding to generate human-like speech with context-aware prosody and intonation.

### Parameters
- `text`: The text to be converted to speech.
- `voice_id`: (Optional) The identifier for the desired voice.
- `speed`: (Optional) The rate of speech.
- `pitch`: (Optional) The pitch of the voice.

## Kotlin Examples

### Basic Usage
```kotlin
val ttsSkill = GeminiTTSSkill()
val audioOutput = ttsSkill.speak("Hello, how can I assist you today?")
```

### Advanced Configuration
```kotlin
val customSpeech = ttsSkill.generate(
    text = "Welcome to the agent library.",
    voiceId = "en-US-Standard-C",
    speed = 1.1,
    pitch = 0.0
)
```
