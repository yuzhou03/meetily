# Audio Processing: Echo Cancellation & Noise Reduction

This document provides a detailed technical overview of how Meetily handles audio
echo cancellation (AEC) and noise reduction (NR) within its audio processing pipeline.

> **Last updated based on source code at:** `frontend/src-tauri/src/audio/`

---

## Table of Contents

- [1. Architecture Overview](#1-architecture-overview)
- [2. Echo Cancellation (AEC)](#2-echo-cancellation-aec)
- [3. Noise Reduction](#3-noise-reduction)
- [4. Audio Processing Pipeline](#4-audio-processing-pipeline)
- [5. Key Components & Implementation Details](#5-key-components--implementation-details)
- [6. Cross-Platform Differences](#6-cross-platform-differences)
- [7. Configuration Options & Switches](#7-configuration-options--switches)
- [8. Audio Mixing Strategy](#8-audio-mixing-strategy)
- [9. Voice Activity Detection (VAD)](#9-voice-activity-detection-vad)
- [10. Summary & Recommendations](#10-summary--recommendations)

---

## 1. Architecture Overview

Meetily's audio system is implemented in Rust within the Tauri backend
(`frontend/src-tauri/src/audio/`). The pipeline is divided into the following
major subsystems:

| Subsystem | Source File(s) | Responsibility |
|-----------|---------------|----------------|
| Capture | `pipeline.rs` (`AudioCapture`), `capture/` | Device capture, resampling, enhancement |
| Mixing | `pipeline.rs` (`AudioMixerRingBuffer`, `ProfessionalAudioMixer`) | Mic + system audio mixing |
| VAD | `vad.rs` (`ContinuousVadProcessor`) | Speech segmentation via Silero VAD |
| Transcription | `transcription/`, `whisper_engine/`, `parakeet_engine/` | STT via Whisper or Parakeet |
| Post-processing | `post_processor.rs` | Transcript text cleanup |

The pipeline processes audio **per-device at capture time** (enhancement) and
**after mixing** (VAD + transcription).

---

## 2. Echo Cancellation (AEC)

### 2.1 Current Status: NOT IMPLEMENTED

**Meetily does not implement Acoustic Echo Cancellation (AEC).**

A thorough analysis of the entire codebase (`frontend/src-tauri/src/audio/`,
`backend/`, and related modules) confirms that:

- There is no AEC algorithm (e.g., adaptive filtering, NLMS, Speex AEC, WebRTC
  AEC) present in the source code.
- No dependency on an AEC library exists in `Cargo.toml`.
- The terms "echo cancellation", "AEC", and "acoustic echo" do not appear
  anywhere in the codebase.

### 2.2 Why AEC Is Not Needed in Meetily's Architecture

Meetily's design inherently avoids the echo problem through **dual-stream
separation**:

```
┌─────────────────────┐    ┌──────────────────────┐
│  Microphone Stream   │    │  System Audio Stream  │
│  (User's voice)      │    │  (Remote participants,│
│  CPAL capture        │    │   media playback)     │
│                      │    │  Core Audio / CPAL     │
└──────────┬───────────┘    └──────────┬────────────┘
           │                           │
           ▼                           ▼
     [Enhancement]              [Raw capture]
     HPF → NR → LUFS            (no enhancement)
           │                           │
           └───────────┬───────────────┘
                       ▼
               [Ring Buffer Mix]
                       │
                       ▼
              [VAD → Transcription]
```

- **Microphone** captures only the local user's voice (plus ambient noise).
- **System audio** captures only remote participants / media playback.
- The two streams are captured from **separate hardware devices** and processed
  independently before being mixed.
- Since the system audio is captured from the OS audio output (not from the
  microphone), there is no acoustic feedback loop that would cause echo.

### 2.3 Potential Scenarios Where Echo Could Occur

Although the architecture avoids echo by design, echo may still be perceived in
these edge cases:

1. **Speaker playback picked up by microphone**: If the user plays audio through
   speakers (not headphones) and the microphone is sensitive enough, the mic
   stream will contain faint system audio bleed-through. This is a physical
   acoustic echo, not a digital one.
2. **Bluetooth device echo**: Bluetooth headsets sometimes introduce latency that
   causes the user's voice to be heard with a delay in their own ear. This is a
   device-level issue, not an application-level echo.

### 2.4 Mitigation Strategies (Without AEC)

- **Recommended: Use headphones** — eliminates acoustic feedback entirely.
- The **RNNoise noise suppressor** (see Section 3) can attenuate low-level
  system audio bleed-through from the microphone.
- The **High-pass filter** (80 Hz cutoff) removes low-frequency rumble that may
  be picked up from speaker vibrations.
- The **EBU R128 normalizer** ensures consistent loudness without amplifying
  background noise excessively.

---

## 3. Noise Reduction

### 3.1 RNNoise-Based Neural Network Noise Suppression

Meetily includes a noise suppression processor based on the **RNNoise** algorithm,
implemented via the [`nnnoiseless`](https://crates.io/crates/nnnoiseless) crate
(version `0.5`).

**Dependency declaration** (`Cargo.toml`):
```toml
# Noise suppression - RNNoise-based neural network noise reduction
nnnoiseless = "0.5"
```

#### 3.1.1 Implementation: `NoiseSuppressionProcessor`

Source: `audio_processing.rs`, struct `NoiseSuppressionProcessor`

```rust
pub struct NoiseSuppressionProcessor {
    denoiser: DenoiseState<'static>,
    frame_buffer: Vec<f32>,
    frame_size: usize,  // 480 samples at 48kHz = 10ms
}
```

**Key characteristics:**

| Property | Value |
|----------|-------|
| Algorithm | RNNoise (recurrent neural network) |
| Library | `nnnoiseless` v0.5 |
| Sample Rate | 48000 Hz (required) |
| Frame Size | 480 samples (10 ms) |
| Noise Reduction | 10–15 dB in typical environments |
| Latency | ~10 ms per frame |
| VAD Output | Returns VAD probability (0.0–1.0) per frame |

#### 3.1.2 Processing Logic

```
Input samples → frame_buffer (accumulate)
                  ↓ (when ≥ 480 samples)
            Extract 480-sample frame
                  ↓
         DenoiseState::process_frame()
                  ↓
        Denoised frame → Output buffer
                  ↓
        Remaining samples stay in buffer
```

- Audio is processed in fixed **480-sample frames** (10 ms at 48 kHz).
- Partial frames are **buffered** for the next call.
- On flush (recording end), remaining samples are **zero-padded** and processed.

#### 3.1.3 Default: DISABLED

**RNNoise is disabled by default.** The controlling flag is:

```rust
// ffmpeg_mixer.rs
pub const RNNOISE_APPLY_ENABLED: bool = false;
```

The rationale stated in the code:

> *"Whisper handles noise well internally — RNNoise is optional"*

This means:
- Whisper's neural network already has strong noise robustness.
- RNNoise may introduce artifacts on clean audio.
- Users who need extra noise suppression can enable it by setting the flag to
  `true` and recompiling.

#### 3.1.4 When RNNoise Is Enabled

If `RNNOISE_APPLY_ENABLED = true`:
- RNNoise is initialized **only for microphone streams** (not system audio).
- It applies as **Step 2** in the enhancement pipeline (after high-pass filter,
  before normalization).
- Health monitoring logs buffered sample count and length deltas every 100 chunks
  to detect latency buildup.

### 3.2 High-Pass Filter

A first-order IIR high-pass filter removes low-frequency rumble below 80 Hz.

```rust
pub struct HighPassFilter {
    alpha: f32,
    prev_input: f32,
    prev_output: f32,
}
```

| Property | Value |
|----------|-------|
| Type | First-order IIR (Infinite Impulse Response) |
| Cutoff | 80 Hz |
| Formula | `y[n] = α × (y[n-1] + x[n] - x[n-1])` |
| Applied to | Microphone only |
| Always on | Yes (when microphone is present) |

This filter removes:
- Desk vibrations and mechanical rumble
- Low-frequency HVAC noise
- Speaker-induced feedback below speech range

### 3.3 Spectral Subtraction (Legacy / Unused)

The codebase contains a `spectral_subtraction()` function and
`average_noise_spectrum()` utility in `audio_processing.rs`. These are **defined
but never called** from the main pipeline. They appear to be legacy
implementations retained for reference.

### 3.4 Whisper's Internal Noise Handling

Whisper (the speech-to-text engine) has inherent noise robustness:
- It was trained on diverse audio including noisy environments.
- It uses attention mechanisms that naturally focus on speech patterns.
- For this reason, the Meetily team chose to disable RNNoise by default.

---

## 4. Audio Processing Pipeline

### 4.1 Complete Pipeline Flow

```
  ┌─────────────────────────────────────────────────────────────┐
  │                   CAPTURE PHASE (per-device)                 │
  │                                                              │
  │  Raw Audio (CPAL/CoreAudio)                                  │
  │       │                                                      │
  │       ▼                                                      │
  │  [1] audio_to_mono()  — Convert multi-channel to mono        │
  │       │                                                      │
  │       ▼                                                      │
  │  [2] Resample to 48kHz (if needed)                           │
  │       │   - Persistent SincFixedIn resampler (rubato)        │
  │       │   - 512-sample buffered chunks                       │
  │       │   - Adaptive quality based on rate ratio             │
  │       │                                                      │
  │       ▼  (Microphone Only)                                   │
  │  [3] HighPassFilter — Remove rumble < 80 Hz                  │
  │       │                                                      │
  │       ▼  (Microphone Only, if RNNOISE_APPLY_ENABLED)         │
  │  [4] NoiseSuppressionProcessor — RNNoise 10-15 dB reduction  │
  │       │                                                      │
  │       ▼  (Microphone Only)                                   │
  │  [5] LoudnessNormalizer — EBU R128 to -23 LUFS              │
  │       │   + True Peak Limiter (-1 dBTP)                      │
  │       │                                                      │
  │       ▼                                                      │
  │  AudioChunk → RecordingState channel                         │
  └─────────────────────────────────────────────────────────────┘
                          │
                          ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                   MIXING PHASE                               │
  │                                                              │
  │  AudioMixerRingBuffer (600ms windows, 4.8s max buffer)       │
  │       │                                                      │
  │       ▼                                                      │
  │  ProfessionalAudioMixer                                      │
  │       │   - Mic at full volume                               │
  │       │   - System at 100% (scaled)                          │
  │       │   - Soft proportional scaling (no hard clipping)     │
  │       │                                                      │
  │       ▼                                                      │
  │  Mixed Audio (48 kHz, mono)                                  │
  └─────────────────────────────────────────────────────────────┘
                          │
                          ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                   VAD PHASE                                  │
  │                                                              │
  │  Resample 48kHz → 16kHz (anti-aliased downsampling)          │
  │       │                                                      │
  │       ▼                                                      │
  │  Silero VAD (30ms chunks = 480 samples at 16kHz)             │
  │       │   - positive_speech_threshold: 0.50                   │
  │       │   - negative_speech_threshold: 0.35                   │
  │       │   - redemption_time: 400 ms                           │
  │       │   - min_speech_time: 250 ms                           │
  │       │   - pre_speech_pad: 300 ms                            │
  │       │   - post_speech_pad: 400 ms                           │
  │       │                                                      │
  │       ▼                                                      │
  │  SpeechSegment (≥ 50ms / 800 samples at 16kHz)               │
  └─────────────────────────────────────────────────────────────┘
                          │
                          ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                   TRANSCRIPTION PHASE                         │
  │                                                              │
  │  Whisper / Parakeet / Deepgram                                │
  │       │   - Internal noise robustness                        │
  │       │   - Language-specific models                         │
  │       │                                                      │
  │       ▼                                                      │
  │  Raw Text → PostProcessor (dedup, artifact removal)           │
  └─────────────────────────────────────────────────────────────┘
```

### 4.2 Processing Order Rationale

The microphone enhancement order is critical:

```
High-Pass Filter → RNNoise → EBU R128 Normalizer
```

1. **High-pass first**: Removes low-frequency energy that would confuse the
   noise suppressor and waste the normalizer's dynamic range.
2. **RNNoise second**: Suppresses background noise before normalization
   prevents the normalizer from amplifying noise.
3. **Normalization last**: Adjusts loudness to broadcast standard (-23 LUFS)
   after the audio is clean.

---

## 5. Key Components & Implementation Details

### 5.1 RNNoise (`NoiseSuppressionProcessor`)

| File | `audio_processing.rs` (lines 227–341) |
|------|---------------------------------------|
| Struct | `NoiseSuppressionProcessor` |
| Crate | `nnnoiseless` v0.5 |
| Core API | `DenoiseState::new()`, `DenoiseState::process_frame()` |

```rust
// Frame-based processing
while self.frame_buffer.len() >= self.frame_size {
    let frame: Vec<f32> = self.frame_buffer.drain(0..self.frame_size).collect();
    let mut denoised_frame = vec![0.0f32; self.frame_size];
    let _vad_prob = self.denoiser.process_frame(&mut denoised_frame, &frame);
    output.extend_from_slice(&denoised_frame);
}
```

### 5.2 High-Pass Filter (`HighPassFilter`)

| File | `audio_processing.rs` (lines 343–403) |
|------|---------------------------------------|
| Type | First-order IIR |
| Cutoff | 80 Hz (configurable at construction) |

```rust
// IIR formula
let filtered = self.alpha * (self.prev_output + sample - self.prev_input);
```

### 5.3 EBU R128 Loudness Normalizer (`LoudnessNormalizer`)

| File | `audio_processing.rs` (lines 139–225) |
|------|---------------------------------------|
| Standard | EBU R128 |
| Target | -23 LUFS |
| True Peak Limit | -1 dBTP |
| Analysis Chunk | 512 samples |
| Library | `ebur128` v0.1 |
| Limiter | 10 ms lookahead true peak limiter |

### 5.4 Resampler (`rubato` SincFixedIn)

| Parameter | Value Based on Ratio |
|-----------|---------------------|
| Ratio ≥ 2.0x | sinc_len=512, Cubic, oversample=512 |
| Ratio ≥ 1.5x | sinc_len=384, Cubic, oversample=384 |
| Ratio > 1.0x | sinc_len=256, Linear, oversample=256 |
| Ratio ≤ 0.5x | sinc_len=512, Cubic, oversample=512 |
| Other | sinc_len=384, Linear, oversample=384 |

Window function: `BlackmanHarris2`, f_cutoff: 0.95.

---

## 6. Cross-Platform Differences

### 6.1 Audio Capture Backends

| Platform | Microphone | System Audio |
|----------|-----------|--------------|
| **macOS** | CPAL | Core Audio (`cidre` crate) or ScreenCaptureKit |
| **Windows** | CPAL (WASAPI) | CPAL (WASAPI loopback) |
| **Linux** | CPAL (PulseAudio/PipeWire) | CPAL (PulseAudio monitor) |

macOS has a dedicated `core_audio.rs` implementation that uses Apple's
Core Audio aggregate device + tap API for low-latency system audio capture.

### 6.2 System Audio Backend Selection

On macOS, users can choose between:
- **Core Audio** (default, recommended): Direct hardware tap via `cidre` crate
- **ScreenCaptureKit**: Apple's screen capture API (includes audio)

On Windows and Linux, only CPAL-based capture is available.

### 6.3 Device Detection

The `device_detection.rs` module provides a 3-layer detection strategy:

1. **Platform-native APIs** (highest accuracy): macOS IOKit, Windows
   DeviceTopology, Linux ALSA
2. **Name-based heuristics**: Matches device names against known Bluetooth
   patterns (e.g., "AirPods", "WH-1000XM4")
3. **Buffer size analysis**: Infer device type from reported buffer sizes

Device kind affects buffer timeouts:

| Device Kind | Min Timeout | Max Timeout |
|-------------|------------|-------------|
| Wired | 20 ms | 50 ms |
| Bluetooth | 80 ms | 200 ms |
| Unknown | 80 ms | 180 ms |

### 6.4 VAD Redemption Time

```rust
let redemption_time = if cfg!(target_os = "macos") { 400 } else { 400 };
```

Currently, the VAD redemption time is set to **400 ms** on all platforms.
The conditional is retained for future platform-specific tuning.

### 6.5 Noise Reduction & Enhancement

The RNNoise noise suppression, high-pass filter, and EBU R128 normalizer are
**identical across all platforms** — they operate on the same 48 kHz mono audio
regardless of OS. The only platform difference is in how the raw audio is
captured before enhancement.

### 6.6 GPU Acceleration for Transcription

| Platform | Default GPU Backend |
|----------|-------------------|
| macOS | Metal + CoreML |
| Windows | CPU (OpenBLAS optional) |
| Linux | CPU (CUDA/Vulkan/ROCm optional) |

GPU acceleration affects Whisper/Parakeet transcription speed, not the audio
enhancement pipeline.

---

## 7. Configuration Options & Switches

### 7.1 Compile-Time Flags

| Flag | File | Default | Description |
|------|------|---------|-------------|
| `RNNOISE_APPLY_ENABLED` | `ffmpeg_mixer.rs` | `false` | Enable/disable RNNoise noise suppression |

To enable RNNoise, change the flag and recompile:
```rust
pub const RNNOISE_APPLY_ENABLED: bool = true;
```

### 7.2 Audio Processing Parameters

| Parameter | Value | Location |
|-----------|-------|----------|
| Target sample rate | 48000 Hz | `pipeline.rs` |
| High-pass cutoff | 80 Hz | `pipeline.rs` |
| EBU R128 target | -23 LUFS | `audio_processing.rs` |
| True peak limit | -1 dBTP | `audio_processing.rs` |
| Normalizer analysis chunk | 512 samples | `audio_processing.rs` |
| RNNoise frame size | 480 samples (10ms) | `audio_processing.rs` |
| Ring buffer window | 600 ms | `pipeline.rs` |
| Ring buffer max | 4800 ms (8× window) | `pipeline.rs` |

### 7.3 VAD Parameters

| Parameter | Value | Location |
|-----------|-------|----------|
| VAD sample rate | 16000 Hz | `vad.rs` |
| VAD chunk size | 480 samples (30ms) | `vad.rs` |
| Positive speech threshold | 0.50 | `vad.rs` |
| Negative speech threshold | 0.35 | `vad.rs` |
| Redemption time | 400 ms | `vad.rs` / `pipeline.rs` |
| Min speech time | 250 ms | `vad.rs` |
| Pre-speech pad | 300 ms | `vad.rs` |
| Post-speech pad | 400 ms | `vad.rs` |
| Minimum segment length | 800 samples (50ms at 16kHz) | `pipeline.rs` |

### 7.4 Mixing Parameters

| Parameter | Value | Location |
|-----------|-------|----------|
| System audio scale | 1.0 (100%) | `pipeline.rs` |
| Mic scale | 0.8 (reserved) | `pipeline.rs` |
| Mixing window | 600 ms | `pipeline.rs` |
| Adaptive ducking (FFmpeg mixer) | Enabled | `ffmpeg_mixer.rs` |
| Speech threshold (ducking) | RMS > 0.01 | `ffmpeg_mixer.rs` |
| System ducking level | 0.60 (60%) | `ffmpeg_mixer.rs` |

---

## 8. Audio Mixing Strategy

### 8.1 Ring Buffer Mixing (Production Pipeline)

The `AudioMixerRingBuffer` in `pipeline.rs` is the active mixing implementation:

1. **Accumulation**: Mic and system samples are added to separate `VecDeque`
   buffers.
2. **Window extraction**: 600 ms windows are extracted when either buffer has
   sufficient data.
3. **Zero-padding**: Incomplete buffers are padded with silence (0.0) rather
   than repeating the last sample.
4. **Soft scaling**: If the mixed sum exceeds ±1.0, proportional scaling is
   applied instead of hard clipping:
   ```rust
   let mixed_sample = if sum_abs > 1.0 {
       sum / sum_abs  // Proportional scaling
   } else {
       sum
   };
   ```

### 8.2 FFmpeg-Style Adaptive Mixer (Alternative)

The `FFmpegAudioMixer` in `ffmpeg_mixer.rs` provides an alternative mixing
strategy with:
- **Per-source buffering** with device-aware timeouts
- **Gap detection** for Bluetooth jitter
- **RMS-based adaptive ducking**: System audio is ducked to 60% when mic speech
  is detected (RMS > 0.01), and restored to 100% when mic is silent.

This mixer is available but the production pipeline currently uses the ring
buffer approach.

---

## 9. Voice Activity Detection (VAD)

### 9.1 Silero VAD

The `ContinuousVadProcessor` uses [Silero VAD](https://github.com/snakers4/silero-vad)
(via `silero_rs` crate) to segment speech from mixed audio.

Key design decisions:
- Operates at **16 kHz** (Silero requirement)
- Processes in **30 ms chunks** (480 samples)
- Returns complete speech segments (not individual frames)
- Minimum segment duration: **250 ms** (prevents Whisper hallucinations)
- Segments shorter than **50 ms** (800 samples) are dropped

### 9.2 VAD and Noise Interaction

The VAD operates **after** all noise processing:
- If RNNoise is enabled, noise-suppressed audio reaches VAD, resulting in
  cleaner speech/silence classification.
- If RNNoise is disabled (default), raw normalized audio reaches VAD, which
  may include background noise but Whisper's robustness compensates.

---

## 10. Summary & Recommendations

### 10.1 Current State

| Feature | Status |
|---------|--------|
| Acoustic Echo Cancellation (AEC) | **Not implemented** |
| RNNoise Noise Suppression | **Implemented, disabled by default** |
| High-Pass Filter (80 Hz) | **Implemented, always on (mic only)** |
| EBU R128 Normalization | **Implemented, always on (mic only)** |
| True Peak Limiting | **Implemented, always on** |
| Spectral Subtraction | **Defined but unused (legacy)** |
| Voice Activity Detection | **Implemented, always on** |

### 10.2 Design Philosophy

Meetily prioritizes a **clean capture → enhance → transcribe** approach:
- Capture separate streams to avoid echo at the architecture level
- Apply minimal enhancement (HPF + optional NR + normalization)
- Rely on Whisper's internal noise robustness for transcription quality
- Avoid over-processing that could introduce artifacts

### 10.3 Enabling Additional Noise Reduction

If you experience excessive background noise:

1. **Enable RNNoise**: Set `RNNOISE_APPLY_ENABLED = true` in `ffmpeg_mixer.rs`
   and rebuild. This provides 10–15 dB noise reduction with minimal latency.
2. **Use headphones**: Eliminates acoustic feedback from speakers.
3. **Select a directional microphone**: Reduces ambient noise at the source.

### 10.4 Future Considerations

The `audio_v2/` module contains placeholder implementations for a next-generation
audio system including:
- Professional audio mixing with dynamic ducking and crossfading
- EBU R128 normalization (refactored)
- True peak limiting (refactored)
- Modular architecture (`ModernAudioSystem`)

This module is currently in **development/placeholder state** (many methods
contain `TODO` comments) and is not used in the production pipeline.
