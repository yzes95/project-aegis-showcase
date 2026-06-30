# 🛡️ Aegis: Digital Shield
### *An intelligent, AI-powered Android accountability app — built to help users break free from porn addiction.*

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
  <img src="https://img.shields.io/badge/React_Native-CLI-61DAFB?style=for-the-badge&logo=react&logoColor=black"/>
  <img src="https://img.shields.io/badge/Kotlin-Native-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white"/>
  <img src="https://img.shields.io/badge/TensorFlow_Lite-AI-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white"/>
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=for-the-badge&logo=postgresql&logoColor=white"/>
</p>

> ⚠️ **Note:** The full source code is maintained in a **private repository**. This showcase repository documents the architecture, engineering decisions, and development lifecycle for portfolio purposes. Contact details are at the bottom.

---

## 📌 Project Overview

Aegis: Digital Shield is a **high-commitment Android accountability application** built for users who want real, inescapable control over their digital habits. Unlike simple blocklists or browser filters that can be disabled in seconds, Aegis is designed to be **resilient by architecture** — it fights back.

### The Three Core Pillars

| Pillar | Description |
|---|---|
| 🔍 **Detect** | Real-time on-device AI scanning + text analysis to identify NSFW content across any app |
| 🔒 **Lock** | Suspends the offending app immediately and forces the user to the home screen |
| 🛡️ **Persist** | Prevents uninstallation, disabling, or bypassing during an active accountability challenge |

---

## 🚦 Current Status

| Phase | Name | Status |
|---|---|---|
| 1 | UI/UX Prototype | ✅ Complete |
| 2 | Native Engine & Detection | ✅ Complete |
| 3 | Self-Defense & Backend | 🚧 In Progress |
| 4 | AI Tier 2 Expansion | 📋 Planned |
| 5 | iOS Expansion & UI Polish | 📋 Planned |

---

## 🏗️ System Architecture

The following diagram shows how the three major layers of Aegis communicate with each other at runtime:

```mermaid
graph TD
    subgraph UI_Layer ["📱 UI Layer (React Native)"]
        A[User Sets Challenge] --> B[DeviceAdminModule]
        B --> C[Requests Device Admin Privilege]
    end

    subgraph Native_Layer ["⚙️ Native Android Layer (Kotlin)"]
        D[AegisAccessibilityService] --> E{Event Router}
        E -->|Settings Package| F["🛡️ handleSelfDefense()\n OEM Multi-Pattern Matcher"]
        E -->|Browser/App Package| G[checkForBadUrls\nText & URL Scanner]
        E -->|Timer Pulse| H["scanScreenContent()\nScreenshot Capture"]

        H --> I[AegisDetectionService\nTensorFlow Lite Engine]
        I --> J{AI Score Check}
        J -->|Score > 0.7| K["🔥 NSFW Detected\n(Broadcast Block)"]
        J -->|Score = 0.0 + Text Match| L["🔥 Incognito Detected\n(Nuke Sequence)"]

        G -->|Nuclear Keyword Found| M[Request Strict Verification]
        M --> H
    end

    subgraph Block_Engine ["🔒 Block Engine"]
        K --> N[blockApp]
        L --> N
        N --> O{Is Browser?}
        O -->|Yes - Normal Tab| P[about:blank → Home → Suspend]
        O -->|Yes - Incognito| Q[4× Back → about:blank → Home → Suspend]
        O -->|No - Regular App| R[4× Back → Home → Suspend]

        F --> S[3× Back → Home\nNo Penalty Applied]
    end

    subgraph Backend ["☁️ Backend (FastAPI + PostgreSQL)"]
        T[Auth: Google OAuth / Email+JWT]
        U[Challenge State Sync]
        V[Progress & Streak Tracking]
    end

    B --> T
    A --> U
```

---

## ⚙️ Technical Highlights

### 🧠 1. On-Device AI — TensorFlow Lite Multi-Crop Scanner
Rather than sending screenshots to a cloud server (a major privacy concern), Aegis runs a **TensorFlow Lite model entirely on the device**.

To maximize detection accuracy, it doesn't just analyze the full screen — it slices the screenshot into **8 distinct regions**:

| Region | Purpose |
|---|---|
| Full Screen | Overall context |
| Center Square | Primary content area |
| Top Half | Header/banner images |
| Bottom Half | Footer thumbnails |
| Top-Left Quadrant | Grid image detection |
| Top-Right Quadrant | Grid image detection |
| Bottom-Left Quadrant | Grid image detection |
| Bottom-Right Quadrant | Grid image detection |

**Why 8 regions?** A single thumbnail of explicit content in a corner would score low on a full-screen analysis. Multi-crop ensures nothing hides in the edges.

---

### 🕵️ 2. The Incognito Nuke Sequence
A key engineering challenge: Android's `FLAG_SECURE` blocks screenshots in Incognito tabs, returning a completely **pitch-black image** — which normally scores `0.0000` (safe!) on the AI model.

**The solution — a 3-step synergy:**
1. **Pure Black Pixel Check:** The Accessibility Service checks 5 pixels (4 corners + center). If all are `#000000`, it sets `isCurrentlyIncognito = true`.
2. **AI + Text Synergy:** If the AI scores `0.0000` **AND** the Text Scanner found a nuclear keyword (e.g. a known NSFW brand name), Aegis concludes: *"Incognito Porn Session Detected."*
3. **The Nuke:** 4 rapid Back taps (destroys incognito history) → `about:blank` injection → Home → App Suspend.

---

### 🛡️ 3. Aegis Self-Defense (OEM Multi-Pattern Matching)
A determined user might try to disable Aegis by navigating to Settings. The Self-Defense system intercepts this at the OS level.

**The challenge:** Android OEMs use different class names for the same settings screens. For example:
- Stock Android: `InstalledAppDetails`
- Samsung One UI: `ApplicationsDetailsActivity`
- Xiaomi MIUI: `ApplicationsDetailsActivity` (different package)
- OnePlus: `AppDetailFragment`

**The solution:** A two-layer detection system:
- **Layer 1 (Fast):** Match against a set of 13 known class name patterns across 7 OEM skins.
- **Layer 2 (Fallback):** Scan the visible node tree for the text `"AegisApp"` — works on any unknown OEM.

When triggered: **3× Back → Home** (no app penalty, just a silent redirect).

---

### 🏃 4. Race Condition Prevention
When an NSFW page is detected, three independent systems may trigger simultaneously:
- The JavaScript layer (URL pattern matching)
- The Native AI (image score)
- The Text Scanner (nuclear keyword match)

Without protection, this caused Chrome to open **3–4 redundant `about:blank` tabs**. The fix: an `isAlreadyBlocked` flag checked atomically before any block sequence executes. Only the **first** trigger wins; all others are silently discarded.

---

## 🔄 SDLC — Development Lifecycle

This project follows an **Iterative & Incremental** SDLC model. Each phase delivers a fully working, testable increment that builds on the previous one.

```mermaid
gantt
    title Aegis: Digital Shield — Development Timeline
    dateFormat  YYYY-MM
    axisFormat  %b %Y

    section Phase 1
    UI/UX Prototype (React Native)     :done, p1, 2025-09, 2025-11

    section Phase 2
    Native Detection Engine (Kotlin + TFLite)   :done, p2, 2025-11, 2026-06

    section Phase 3
    Self-Defense + Backend (FastAPI)   :active, p3, 2026-06, 2026-09

    section Phase 4
    AI Tier 2 Model Upgrade            :p4, 2026-09, 2026-12

    section Phase 5
    iOS Expansion + UI Polish          :p5, 2027-01, 2027-06
```

### Why Iterative & Incremental?
Early in the project, the full technical requirements were unknown. For example:
- The original plan was a **VPN-based network interceptor** → discovered during Phase 2 implementation that the Accessibility Service approach was faster, more reliable, and didn't require root.
- The original lockdown used a **full-screen overlay** → testing revealed Android restrictions, so we pivoted to **app suspension**.
- **Incognito tab vulnerabilities** were only discovered during real-world testing of Phase 2.

Each discovery fed directly into the next iteration. A rigid Waterfall approach would have required scrapping and restarting multiple times.

---

## 📊 Commit Highlights

| Commit | Phase | Description |
|---|---|---|
| `5551ed9` | Phase 1 | Initial README and project documentation |
| `7c9163a` | Phase 1 | UI/UX prototype — screens, modals, and navigation |
| `65a810c` | Phase 2 | Device Admin privilege request and Accessibility Service setup |
| `7e1b53f` | Phase 2 | TFLite screen capture algorithm and multi-crop detection design |
| `cc30df4` | Phase 2 | Major: Fixed detection logic, Incognito Nuke Sequence, race condition fix, dynamic browser tracking |
| `c65a867` | Phase 2 | `.gitignore` hardening for native build artifacts |
| `6afbda2` | Phase 3 | README rewrite to reflect actual Phase 2 architecture |
| `3388340` | Phase 3 | Phase 3: OEM-aware Self-Defense settings interceptor + removed `lockNow()` vulnerability |

> 📁 Full commit history is available in the private repository. Contact me for access.

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|---|---|---|
| UI Framework | React Native (CLI) | Cross-platform foundation with direct native module bridge |
| Native Layer | Kotlin (Android) | Required for Accessibility Service, Device Admin, and TFLite integration |
| AI/ML | TensorFlow Lite | On-device inference — no cloud, no privacy risk |
| Backend API | FastAPI (Python) | High-performance async REST API for auth and data sync |
| Database | PostgreSQL | Robust relational schema for user progress and challenge state |
| Auth | Google OAuth + JWT | Flexible dual-auth with token-based session verification |
| Version Control | Git + GitHub | Branch-per-phase strategy with linked GitHub Project Boards |

---

## 📁 Repository & Project Board Structure

The private repository is organized with dedicated branches and GitHub Project Boards per phase:

| Branch | Project Board | Status |
|---|---|---|
| `phase1` | Phase 1: UI Prototype | ✅ Closed |
| `phase2` | Phase 2: Lock Down Algorithm | ✅ Closed |
| `phase3` | Phase 3: Aegis Self-Defense & Zero Tolerance | 🚧 Open |

---

## 👤 About the Developer

**Yahya ZE Salman**
Back-End Developer | AI/ML Developer | Cyber Security Seeker

This project represents a full-stack, cross-discipline engineering challenge — combining mobile UI, native Android development, on-device machine learning, REST API design, and security hardening into a single production-grade application.

📬 **Interested in the full source code or a deeper walkthrough?**
Reach out via GitHub: [@yzes95](https://github.com/yzes95)

---

<p align="center"><sub>🛡️ Aegis: Digital Shield — Built with purpose.</sub></p>
