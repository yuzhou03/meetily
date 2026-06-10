# Meetily Features & Capabilities Overview

A comprehensive reference for Meetily's core features, technical capabilities, strengths, limitations, differentiation, and target audience.

## Core Features & Technical Capabilities

### Real-Time Local Transcription

- Captures and transcribes meetings in real time using **Whisper** or **Parakeet** speech-to-text models.
- All transcription runs **entirely on-device** — no audio is ever sent to external servers.
- Supports multiple languages for transcription.

### AI-Powered Meeting Summaries

- Generates structured meeting summaries using configurable LLM backends.
- Supported providers:
  - **Ollama** (local, recommended for privacy)
  - **Claude** (Anthropic)
  - **GPT-4** (OpenAI-compatible)
  - **Groq** (fast inference)
  - **OpenRouter** (multi-model gateway)
  - **Custom OpenAI-compatible endpoints** (self-hosted or third-party)

### Professional Audio Engine

- Simultaneous **microphone and system audio** capture.
- Intelligent audio ducking and clipping prevention.
- Professional-grade audio mixing for clear multi-speaker recordings.

### Import & Enhance (Beta)

- Import existing audio files to generate transcripts.
- Re-transcribe previously recorded meetings with a different model or language.
- All import and enhancement processing is performed locally.

### GPU Acceleration

Hardware acceleration is built in and automatically enabled at build time:

| Platform | Acceleration |
|----------|-------------|
| macOS (Apple Silicon) | Metal + CoreML |
| Windows / Linux (NVIDIA) | CUDA |
| Windows / Linux (AMD/Intel) | Vulkan |
| No GPU available | Optimized CPU fallback |

### Flexible Data Persistence

- Local **SQLite** database for meeting metadata, transcripts, and summaries.
- All data — models, recordings, transcripts, summaries — stored on-device.
- Database import/export support for backup and migration.

### Cross-Platform Support

- **macOS** — Apple Silicon and Intel, distributed as `.dmg`.
- **Windows** — x64, distributed as `.exe` installer.
- **Linux** — Built from source with automatic GPU detection.

### Open Source & Extensible

- MIT-licensed, fully open source.
- Tauri-based architecture allows Rust and web developers to contribute independently.
- Custom summary templates and workflows (Meetily PRO).

---

## Key Advantages (Pros)

1. **Complete Data Sovereignty** — Zero data leaves the device. Ideal for regulated industries (defense, healthcare, legal, finance).
2. **No Recurring API Costs** — Uses free, open-source AI models (Whisper, Parakeet, Ollama) instead of expensive cloud APIs.
3. **Offline-First** — Fully functional without an internet connection when using local models.
4. **Privacy by Design** — No telemetry, no cloud sync, no third-party data access.
5. **Cost-Effective at Scale** — No per-seat or per-minute pricing; run on your own hardware.
6. **Hardware Acceleration** — Automatic GPU detection maximizes transcription speed on supported hardware.
7. **Multi-Provider Flexibility** — Switch between local and cloud AI providers without changing workflows.
8. **Professional Audio Quality** — Dual-channel capture (mic + system) with intelligent mixing.
9. **Lightweight Desktop App** — Tauri framework produces a small binary with low memory footprint compared to Electron-based alternatives.
10. **Active Open-Source Community** — Regular contributions, transparent development, and community-driven feature roadmap.

---

## Limitations & Areas for Improvement (Cons)

1. **No Speaker Diarization (Community Edition)** — Cannot automatically distinguish between different speakers. *(Planned for PRO mid-June.)*
2. **Build-from-Source on Linux** — No pre-built Linux binaries; requires manual compilation with Rust and Node.js toolchains.
3. **Local Model Quality** — Open-source Whisper/Parakeet models may produce lower accuracy than commercial cloud APIs for certain accents or noisy environments.
4. **Single-User Focus** — Community Edition is designed for individual use; team collaboration features are limited to PRO.
5. **No Calendar Integration (Yet)** — Meetings must be started manually; automatic calendar sync is planned for PRO.
6. **No Chat-with-Meeting Feature (Yet)** — Querying meeting content via natural language is not yet available in the Community Edition.
7. **GPU Recommended for Best Performance** — CPU-only mode works but can be significantly slower for long meetings or high-quality models.
8. **Export Formats** — Community Edition supports Markdown export; advanced formats (PDF, DOCX) are PRO-only.

---

## Differentiation from Competing Solutions

| Aspect | Meetily | Cloud-Based Alternatives | Other Open-Source Tools |
|--------|---------|--------------------------|-------------------------|
| **Data Location** | 100% on-device | Vendor servers | Varies |
| **Transcription Engine** | Local Whisper / Parakeet | Proprietary cloud APIs | Often require external APIs |
| **Offline Support** | Full offline capability | Requires internet | Partial |
| **Cost** | Free (MIT license) | Per-seat / per-minute pricing | Free, but may need paid APIs |
| **GPU Acceleration** | Built-in, auto-detected | N/A (cloud) | Manual configuration |
| **AI Summary Providers** | 6+ providers + custom endpoints | Single vendor | Limited |
| **App Size** | Lightweight (Tauri) | N/A (web-based) | Varies (often Electron) |
| **Privacy Compliance** | Zero data exposure | Vendor-dependent | Depends on implementation |

### Key Differentiators

- **Privacy-First Architecture** — Unlike Otter.ai, Fireflies.ai, or Microsoft Copilot, Meetily never uploads audio or transcripts to external servers.
- **Self-Contained Desktop App** — A single installable binary with no external dependencies or browser requirements.
- **Model Flexibility** — Users choose their transcription model and LLM provider, avoiding vendor lock-in.
- **Enterprise-Ready Privacy** — Designed for GDPR, HIPAA, and defense-sector compliance where cloud tools are disallowed.

---

## Target Use Cases & Audience

### Primary Audience

- **Privacy-conscious professionals** — Lawyers, doctors, consultants, and executives handling sensitive discussions.
- **Enterprise teams** — Organizations with strict data governance, compliance, or air-gapped requirements.
- **Developers and engineers** — Technical users who prefer open-source tools and local-first workflows.
- **Researchers and academics** — Individuals who need reliable transcription without data-sharing agreements.

### Ideal Use Cases

| Scenario | Why Meetily |
|----------|-------------|
| Legal consultations | Attorney-client privilege requires zero data exposure |
| Healthcare meetings | HIPAA compliance demands local-only processing |
| Defense & government | Air-gapped environments with no internet access |
| Board meetings | Sensitive corporate strategy discussions |
| Client interviews | Confidential information must not leave the device |
| Academic research | IRB requirements for data handling |
| Personal productivity | Cost-free, always-available meeting notes |

### When to Consider Meetily PRO

- Need **speaker diarization** to separate multiple participants.
- Require **advanced export formats** (PDF, DOCX) for formal documentation.
- Want **custom summary templates** tailored to specific workflows.
- Need **auto-meeting detection** for hands-free recording.
- Operating in a **team or self-hosted deployment** scenario (2–100 users).

---

## Related Documentation

- [System Architecture](../architecture.md) — Detailed component and data-flow overview.
- [Building from Source](../BUILDING.md) — Step-by-step build instructions for all platforms.
- [GPU Acceleration](../GPU_ACCELERATION.md) — Hardware acceleration setup guide.
- [Privacy Policy](../../PRIVACY_POLICY.md) — Full privacy policy and data handling details.
