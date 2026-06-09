---
name: gemini-tts
description: >-
  This skill is used to implement Android text-to-speech features in Kotlin using Gemini API text-to-speech (TTS) models.
  Use it when you need to convert text into voice outputs, configure single/multi-speaker voices, steer speech styles with natural language or
  inline tags, and play back or process the resulting PCM audio on Android devices.
---

# Gemini Text-to-Speech (TTS)

Use this skill to implement high-quality, controllable text-to-speech capabilities in Android applications using Kotlin and the Google Gen AI client library.

## Core Principles

1.  **Controllable Speech**: Steer the accent, tone, pace, and style of the voice using natural language prompting or inline audio tags.
2.  **Android Integration**: Optimize and format code examples for native Android platforms, utilizing standard Kotlin types.
3.  **Low Latency**: Choose the correct model variant based on the latency requirements of the app (e.g. `gemini-3.1-flash-tts-preview` or `gemini-2.5-flash-preview-tts`).

## Strict Rules for Gemini TTS Mode

1.  **Configure Audio Modality**: Always specify `responseModalities = listOf("AUDIO")` in the generation config.
2.  **Define Speech Config**: Provide a `SpeechConfig` containing a `VoiceConfig` (prebuilt voice name) or a `MultiSpeakerVoiceConfig`.
3.  **Handle Raw PCM Output**: The returned audio is raw signed 16-bit PCM (sample rate 24000 Hz, mono). Educate the developer that this cannot be played directly using standard `MediaPlayer` (as it lacks headers). Show how to play it with `AudioTrack` or wrap it with a WAV header.
4.  **Audio Tags Compliance**: Incorporate inline audio tags like `[excitedly]`, `[whispers]`, `[sighs]`, etc., inside the prompt text.
5.  **Language Fallbacks**: Remind the developer that TTS models detect the language automatically, but English audio tags should be used even for non-English transcripts.

## Voice Options

The Gemini TTS models support 30 prebuilt voices. Common options include:
- **Puck** (Upbeat)
- **Charon** (Informative)
- **Kore** (Firm)
- **Fenrir** (Excitable)
- **Leda** (Youthful)
- **Aoede** (Breezy)
- **Enceladus** (Breathy)
- **Achird** (Friendly)
- **Vindemiatrix** (Gentle)
- **Sulafat** (Warm)

## Kotlin for Android Examples

### 1. Single-Speaker TTS Configuration
```kotlin
import com.google.ai.client.generativeai.GenerativeModel
import com.google.ai.client.generativeai.type.generationConfig
import com.google.ai.client.generativeai.type.SpeechConfig
import com.google.ai.client.generativeai.type.VoiceConfig
import com.google.ai.client.generativeai.type.PrebuiltVoiceConfig
import com.google.ai.client.generativeai.type.BlobPart

val model = GenerativeModel(
    modelName = "gemini-3.1-flash-tts-preview",
    apiKey = YOUR_API_KEY,
    generationConfig = generationConfig {
        responseModalities = listOf("AUDIO")
        speechConfig = SpeechConfig(
            voiceConfig = VoiceConfig(
                prebuiltVoiceConfig = PrebuiltVoiceConfig(
                    voiceName = "Kore"
                )
            )
        )
    }
)

suspend fun generateSpeech(text: String): ByteArray? {
    val response = model.generateContent("Say cheerfully: $text")
    val rawBytes: ByteArray? = response.candidates.firstOrNull()
        ?.content?.parts
        ?.filterIsInstance<BlobPart>()
        ?.firstOrNull()
        ?.data
    return rawBytes
}
```

### 2. Multi-Speaker Conversation Configuration
```kotlin
import com.google.ai.client.generativeai.GenerativeModel
import com.google.ai.client.generativeai.type.generationConfig
import com.google.ai.client.generativeai.type.SpeechConfig
import com.google.ai.client.generativeai.type.VoiceConfig
import com.google.ai.client.generativeai.type.PrebuiltVoiceConfig
import com.google.ai.client.generativeai.type.MultiSpeakerVoiceConfig
import com.google.ai.client.generativeai.type.SpeakerVoiceConfig
import com.google.ai.client.generativeai.type.BlobPart

val multiSpeakerModel = GenerativeModel(
    modelName = "gemini-3.1-flash-tts-preview",
    apiKey = YOUR_API_KEY,
    generationConfig = generationConfig {
        responseModalities = listOf("AUDIO")
        speechConfig = SpeechConfig(
            multiSpeakerVoiceConfig = MultiSpeakerVoiceConfig(
                speakerVoiceConfigs = listOf(
                    SpeakerVoiceConfig(
                        speaker = "Joe",
                        voiceConfig = VoiceConfig(
                            prebuiltVoiceConfig = PrebuiltVoiceConfig(voiceName = "Kore")
                        )
                    ),
                    SpeakerVoiceConfig(
                        speaker = "Jane",
                        voiceConfig = VoiceConfig(
                            prebuiltVoiceConfig = PrebuiltVoiceConfig(voiceName = "Puck")
                        )
                    )
                )
            )
        )
    }
)

suspend fun generateConversationAudio(): ByteArray? {
    val transcript = """
        Joe: How's it going today Jane?
        Jane: [excitedly] Not too bad, how about you?
    """.trimIndent()
    
    val response = multiSpeakerModel.generateContent(transcript)
    return response.candidates.firstOrNull()
        ?.content?.parts
        ?.filterIsInstance<BlobPart>()
        ?.firstOrNull()
        ?.data
}
```

### 3. Playback of Raw PCM Data on Android
```kotlin
import android.media.AudioAttributes
import android.media.AudioFormat
import android.media.AudioTrack

fun playPcmAudio(pcmData: ByteArray) {
    val sampleRate = 24000 // Gemini TTS default sample rate is 24kHz
    
    val audioTrack = AudioTrack.Builder()
        .setAudioAttributes(
            AudioAttributes.Builder()
                .setUsage(AudioAttributes.USAGE_MEDIA)
                .setContentType(AudioAttributes.CONTENT_TYPE_SPEECH)
                .build()
        )
        .setAudioFormat(
            AudioFormat.Builder()
                .setEncoding(AudioFormat.ENCODING_PCM_16BIT)
                .setSampleRate(sampleRate)
                .setChannelMask(AudioFormat.CHANNEL_OUT_MONO)
                .build()
        )
        .setBufferSizeInBytes(pcmData.size)
        .setTransferMode(AudioTrack.MODE_STATIC)
        .build()

    audioTrack.write(pcmData, 0, pcmData.size)
    audioTrack.play()
}
```
