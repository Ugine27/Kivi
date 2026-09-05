# Engineering & Product Plan — Kivi 🗓️

---

## 1. Technical Architecture Overview

Kivi is composed of four modular layers designed for high responsiveness, low memory footprint, and modular integration:

```mermaid
graph TB
    subgraph Client Layer ["Client & Ambient Runtime"]
        HK["Hotkey Daemon (`fn` / Global Listener)"]
        AC["Audio Capture & Ring Buffer"]
        VAD["Voice Activity Detection (VAD)"]
        UI["Overlay HUD & Audio Earcons"]
    end

    subgraph Core Runner Layer ["Kivi Execution Runner"]
        ORCH["Audio Pipeline Orchestrator"]
        RS["Revision Store & History"]
        TR["Task & Action Dispatcher"]
        TEL["PostHog & Diagnostic Telemetry"]
    end

    subgraph Speech & Intelligence Layer ["Speech & Model Engine"]
        STT["Multilingual ASR (Sarvam / Indic Models)"]
        CS["Code-Switching & Dialect Classifier"]
        LLM["Contextual Formatter & Disfluency Filter"]
    end

    subgraph Injection Layer ["OS Integration"]
        AX["macOS Accessibility API (AXUIElement)"]
        EV["Quartz Event Tap / Clipboard Injector"]
    end

    HK --> AC --> VAD --> ORCH
    ORCH --> STT --> CS --> LLM
    LLM --> RS --> TR --> AX & EV
    ORCH -.-> UI
    TR -.-> TEL
```

---

## 2. Release Milestones & Roadmap

```mermaid
gantt
    title Kivi Implementation Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 1: Foundation
    Audio Capture & Hotkey Daemon       :active, p1_1, 2026-09-01, 14d
    VAD & Stream Chunking                :active, p1_2, 2026-09-08, 14d
    Earcon Audio Feedback Dispatcher     :p1_3, 2026-09-15, 7d
    section Phase 2: Speech Pipeline
    Sarvam Indic ASR Integration         :p2_1, 2026-09-22, 21d
    Code-Switching Logic & Dialect Eval  :p2_2, 2026-10-06, 14d
    Contextual Text Formatter (LLM)      :p2_3, 2026-10-13, 14d
    section Phase 3: Desktop Injection
    macOS Accessibility Text Insertion   :p3_1, 2026-10-27, 14d
    Cross-App Focus & History Tracking   :p3_2, 2026-11-03, 14d
    Incognito Mode & Privacy Controls    :p3_3, 2026-11-10, 7d
    section Phase 4: Action Runner
    Kivi Runner & Workflow Engine        :p4_1, 2026-11-17, 21d
    Custom Dictionary & Phonetics        :p4_2, 2026-12-01, 14d
    Telemetry & Continuous Tuning        :p4_3, 2026-12-08, 14d
```

---

## 3. Phase Breakdown & Deliverables

### Phase 1: Core Foundation & Ambient Runtime
- **Audio Capture Subsystem:** 16kHz mono audio capture via CoreAudio / AVAudioEngine with ring-buffer streaming.
- **Voice Activity Detection (VAD):** Low-latency Silero-VAD or WebRTC VAD to detect speech start, pause, and endpointing.
- **System Event Listener:** Low-overhead global event tap for `fn` key press-to-talk and toggle modes.
- **Earcon Feedback:** Seamless audio feedback triggered at state transitions (`start.wav`, `complete.wav`, `error.wav`).

### Phase 2: Speech & Intelligence Pipeline
- **ASR Engine Integration:** Bidirectional streaming connection to high-accuracy multilingual ASR models.
- **Code-Switching Engine:** Real-time decoding capable of transcribing Hindi/Tamil/Telugu interleaved with English syntax.
- **Post-Processing & Formatting:** Context-aware grammar repair, intelligent punctuation, numeral and currency conversion, and removal of filler words.

### Phase 3: Desktop Integration & Injection
- **Universal Text Insertion:** Integration with macOS Accessibility API (`AXUIElementSetAttributeValue`) with fallback to pasteboard simulation for non-AX applications.
- **Context Extraction:** Querying focused window title, active application bundle ID, and nearby surrounding text buffer to adjust tone and structure.
- **Local History & Revisions:** SQLite/CoreData revision log allowing instantaneous `Cmd+Z` undo or quick revision tap.

### Phase 4: Kivi Action Runner
- **Command & Action Dispatcher:** Triggering system actions (open applications, search web, run CLI commands) when intent flags are detected.
- **Custom Dictionary & Personal Lexicon:** User-defined vocabulary list for technical jargon, internal product names, and abbreviations.
- **Telemetry & Diagnostics:** Privacy-first telemetry (PostHog) tracking transcription latency, error rates, and failure recovery.

---

## 4. Technical Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Desktop App Runtime** | Swift / AppKit / SwiftUI | Native macOS performance, minimal battery overhead, direct Metal & CoreAudio access. |
| **Audio Processing** | CoreAudio / AVAudioEngine | Low-latency audio I/O with hardware acceleration. |
| **Speech Models** | Sarvam AI ASR / Whisper / Custom Indic | Industry-leading performance on 22+ Indian languages and code-switched audio. |
| **System Event Tap** | Quartz Event Services (`CGEventTap`) | Reliable system-wide key interception (`fn` key). |
| **Text Injection** | macOS Accessibility (`AXUIElement`) | Non-destructive in-place text modification. |
| **Diagrams & Visuals** | Mermaid.js + WebKit / Metal Renderer | Lightweight inline diagram and UI rendering. |

---

## 5. Risk Assessment & Mitigations

| Risk | Impact | Likelihood | Mitigation Strategy |
| :--- | :---: | :---: | :--- |
| **macOS Accessibility Permissions Friction** | High | High | Clear, onboarding wizard guiding users to grant Accessibility and Input Monitoring permissions. |
| **Network Latency Spikes in Cloud ASR** | High | Medium | Implement local on-device quantized model fallback for offline/low-bandwidth environments. |
| **Text Insertion Incompatibilities (Electron apps)** | Medium | Medium | Hybrid injection: try Accessibility API first; fallback to simulated clipboard paste if unhandled. |
| **False Endpointing during Pauses** | Medium | Low | Dynamic VAD hysteresis based on user speech cadence and sentence context. |
