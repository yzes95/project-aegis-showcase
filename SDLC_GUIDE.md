# SDLC Guide - Aegis: Digital Shield

> **For:** Yahya (Developer) | **Purpose:** Understanding & explaining the project's Software Development Life Cycle to recruiters and interviewers.

---

## 1. What Is an SDLC?

Think of building a house.

You don't just show up with bricks and start laying walls. You first figure out *what* you're building (planning), draw up blueprints (design), actually construct it (implementation), inspect every room for cracks (testing), hand over the keys (deployment), and then maybe fix the leaky tap six months later (maintenance).

**The Software Development Life Cycle (SDLC) is exactly that - but for software.**

It's a structured process that guides how a piece of software is *conceived, built, tested, and maintained*. Every professional software project follows some version of an SDLC, whether they name it or not. Knowing yours - and being able to explain it - is what separates a developer from an *engineer*.

The classic SDLC phases are:

| Phase | What Happens |
|---|---|
| **Planning** | Define the problem, scope, timeline, and feasibility |
| **Design** | Architect the system - UI, data flow, tech stack |
| **Implementation** | Actually write the code |
| **Testing** | Find and fix bugs, validate behaviour |
| **Deployment** | Release to users |
| **Maintenance** | Monitor, patch, improve |

---

## 2. Which SDLC Model Did We Follow?

### The Iterative & Incremental Model

We didn't follow a rigid, one-shot plan. We followed the **Iterative & Incremental model** - which means we build the product in *phases*, each one adding a meaningful slice of functionality, and each one going through its own mini-SDLC loop of plan -> design -> build -> test.

Think of it like building a car:
- **Sprint 1:** A working skateboard (you can move)
- **Sprint 2:** A scooter (you move faster, with direction)
- **Sprint 3:** A motorbike (real power, real control)
- **Sprint 4:** The full car

At each stage, the product *works* and can be evaluated. You're not waiting until everything is finished to discover a problem.

---

### Why Was This Right for Aegis?

1. **Requirements weren't fully known upfront.**
   Nobody builds an NSFW content-blocker on Android and knows exactly how it'll behave on day one. We had to discover things like OEM browser quirks, incognito loophole behaviour, and the limitations of Android's Accessibility Service *by building and testing* - not by theorising.

2. **Heavy testing was required.**
   The core feature - detecting and blocking NSFW content in real-time - needs to be validated repeatedly. The kind of bugs that surface here (race conditions, overlay timing issues, false positives in the AI model) only appear when real content is thrown at a real device. You can't spec-test your way out of that.

3. **Each phase is a foundation for the next.**
   The UI shell (Phase 1) had to exist before the native blocking engine (Phase 2) could be wired in. The blocking engine had to exist before the self-defence layer (Phase 3) could protect it. This dependency chain *forces* an incremental approach.

---

### Why Would Waterfall Have Failed Here?

The **Waterfall model** says: fully plan everything -> fully design everything -> fully build everything -> fully test everything -> release. It's linear and rigid.

For Aegis, that would've been a disaster because:
- We discovered architectural flaws (e.g., the VPN approach didn't work) *during* implementation - not during planning.
- The AI model accuracy for NSFW detection needed live testing to tune thresholds - you can't know the right thresholds before you run real images through the model.
- Requirements evolved (e.g., incognito blocking wasn't in the original scope - it was discovered as a gap during testing).

In short: Waterfall assumes you know everything before you start. We didn't. Nobody building novel software does.

---

## 3. How the Phases Map to the SDLC

| Phase | Name | SDLC Stages Covered | Status |
|---|---|---|---|
| **Phase 1** | UI Prototype | Planning + Design + Implementation (partial) | Complete |
| **Phase 2** | Native Engine | Implementation + Testing + Deployment (internal) | Complete |
| **Phase 3** | Self-Defense + Backend | Implementation + Testing | In Progress |
| **Phase 4** | AI Tier 2 | Planning (partial) - Design not started | Planned |
| **Phase 5** | iOS + UI Polish | Planning - Design not started | Planned |

### What Each Phase Actually Delivered

**Phase 1 - UI Prototype**
- The shell of the React Native app was built.
- Screens were designed and prototyped (onboarding, dashboard, settings).
- This was primarily a **planning + design phase** - establishing the visual contract and architecture before any native code was written.

**Phase 2 - Native Engine**
- The Kotlin-based content detection engine was built and wired into the app.
- TensorFlow Lite was integrated to run on-device NSFW classification.
- The Accessibility Service was hooked up to observe and intercept content.
- Internal testing began - running real NSFW content through the system to validate blocking behaviour.
- This is the heaviest **implementation + testing** phase.

**Phase 3 - Self-Defense + Backend (Current)**
- Building the FastAPI + PostgreSQL backend for user accounts, session tokens, and analytics.
- Implementing self-defence mechanisms so the app cannot be uninstalled or bypassed.
- Implementing the Incognito Nuke Sequence and OEM multi-pattern matching.
- This is a deep **implementation + testing** phase with significant architectural decisions being made.

**Phase 4 - AI Tier 2 (Planned)**
- A second, more sophisticated AI model layer for borderline content classification.
- Requires a fresh **design phase** - model selection, training data strategy, threshold tuning plan.

**Phase 5 - iOS + UI Polish (Planned)**
- Cross-platform expansion and final UX refinement.
- Requires full **planning + design** before implementation begins.

---

## 4. Key SDLC Decisions Made During Development

These are real engineering pivots made during the project - each one is a story you can tell a recruiter.

---

### Decision 1: VPN Approach -> Accessibility Service Approach

**What happened:**
Early in Phase 2, the plan was to intercept network traffic using a local VPN tunnel on the device - a common technique for content filtering. After implementation attempts and research, it became clear this approach had major limitations: it required a persistent VPN connection, struggled with HTTPS traffic (TLS inspection is legally and technically complex), and drained battery significantly.

**The pivot:**
We switched to Android's **Accessibility Service**, which observes on-screen content rather than network packets. This lets us detect NSFW content at the *display* level, regardless of whether it comes from a browser, an app, or a social media feed.

**SDLC lesson:** This is a classic mid-implementation architectural pivot - caught during the **Implementation phase**, evaluated, and resolved with a redesign. Waterfall would've locked us into the wrong approach.

---

### Decision 2: Firebase -> FastAPI + PostgreSQL

**What happened:**
Firebase was the original plan for the backend - quick to set up, handles auth and real-time data. But as the requirements grew (custom analytics, granular session control, JWT-based accountability tokens, complex relational data between users, sessions, and violations), Firebase's NoSQL structure and pricing model became a limiting factor.

**The pivot:**
We built a custom **FastAPI** backend with a **PostgreSQL** relational database. This gives us full control over the data model, query logic, and security posture.

**SDLC lesson:** This decision was made during the **Planning phase of Phase 3** - before committing to Firebase implementation. Catching this in planning rather than post-implementation saved significant rework.

---

### Decision 3: Overlay Screen -> App Suspension

**What happened:**
The first approach to blocking was displaying a full-screen overlay (a `WindowManager` overlay with `TYPE_APPLICATION_OVERLAY`) on top of NSFW content. This *looked* like it was blocking, but power users could dismiss it by switching apps or using gesture navigation.

**The pivot:**
We moved to **app suspension** - when NSFW content is detected, the offending app is put into a suspended or force-stopped state, making it genuinely inaccessible rather than just visually obscured.

**SDLC lesson:** This gap was discovered during **Testing phase** of Phase 2 - real-world user-path testing revealed the overlay bypass. This is exactly why testing against adversarial usage matters.

---

### Decision 4: The Incognito Nuke Sequence

**What happened:**
During testing, it became obvious that a user could bypass all content blocking simply by opening Chrome in incognito mode. Incognito tabs are invisible to Accessibility Services by design - Chrome explicitly blocks content observation in incognito.

**The solution:**
We built the **Incognito Nuke Sequence** - a detection routine that identifies when any browser enters incognito mode (via window title patterns, URL bar changes, and UI node signatures) and immediately closes the incognito session or suspends the app.

**SDLC lesson:** This feature was *not in the original scope*. It was discovered as a gap during **Testing phase** and fed back into the Implementation phase as an unplanned but critical requirement. This is the SDLC working exactly as it should - testing informs implementation.

---

### Decision 5: OEM Multi-Pattern Matching

**What happened:**
During the **Planning phase of Phase 3**, while mapping the self-defence and incognito-blocking strategy, it became clear that different Android OEMs (Samsung, Xiaomi, OnePlus, OPPO, etc.) render the same browser UI differently. A Chrome incognito tab on a Samsung device has different Accessibility node signatures than the same tab on a Xiaomi device.

**The solution:**
We designed an **OEM multi-pattern matching system** - a configurable set of detection patterns per OEM brand, allowing the Incognito Nuke Sequence and other detection logic to work across the Android fragmentation landscape.

**SDLC lesson:** This was caught in **Planning** before implementation began - the best possible time. Research into OEM fragmentation during planning phase prevented a massive testing failure down the line.

---

## 5. How To Explain This to a Recruiter

Here's a confident, natural script you can use. Practice saying this out loud.

> *"Aegis follows an Iterative and Incremental SDLC model - we build the product in phases, each one going through its own cycle of planning, design, implementation, and testing before the next phase begins. This was the right choice because the requirements evolved as we built - for example, we discovered the incognito bypass during testing and had to design a whole detection sequence around it that wasn't in the original plan. We made several significant architectural pivots along the way, like switching from a VPN-based approach to Android's Accessibility Service, and replacing Firebase with a custom FastAPI and PostgreSQL backend - all decisions driven by what we learned during the development process. Right now we're in Phase 3, building the self-defence layer and backend infrastructure, with AI enhancement and iOS expansion planned for Phases 4 and 5."*

---

## 6. Key Engineering Terms to Know

A recruiter might ask you to define these. Here are plain, confident explanations.

---

### Accessibility Service
An Android system API that lets an app observe and interact with the UI of *other apps* on the device - reading text on screen, detecting window changes, and triggering actions. Aegis uses it to watch for NSFW content across any app without needing network access. It requires explicit user permission and is a privileged capability.

---

### TensorFlow Lite
Google's lightweight version of the TensorFlow machine learning framework, designed to run directly on mobile devices (on-device inference). Rather than sending images to a server for NSFW classification, TensorFlow Lite runs the model *on the device itself* - faster, private, and works offline. This is the AI core of Aegis's detection engine.

---

### Device Admin API
An Android API that grants an app elevated device management permissions - including the ability to enforce policies, prevent uninstallation, and lock the device. Aegis uses this as part of its self-defence layer to prevent the user (or a bypasser) from simply deleting the app.

---

### FLAG_SECURE
An Android window flag that prevents a window's content from being captured by screenshots or screen recording, and also blocks it from appearing in the Recents/Overview screen. Aegis sets this flag on its own screens to prevent bypass via screen-capture tools.

---

### Race Condition
A bug that occurs when two processes run simultaneously and the outcome depends on which one finishes first - in a way the developer didn't intend. In Aegis, a race condition could occur if the NSFW detection engine fires at the same time as the blocking overlay is being constructed - resulting in the content being briefly visible before the block activates. Solving race conditions requires careful thread synchronisation.

---

### OEM
*Original Equipment Manufacturer* - in the Android world, this refers to the hardware makers who put their own skin on top of Android (Samsung's One UI, Xiaomi's MIUI, OnePlus's OxygenOS, etc.). Each OEM modifies the Android UI layer differently, which means the same browser looks different at the Accessibility node level on different devices. Aegis accounts for this with multi-pattern matching.

---

### JWT Token
*JSON Web Token* - a compact, digitally signed token used for authentication. When a user logs into Aegis's backend (FastAPI), they receive a JWT token. Every subsequent API request includes this token so the server can verify who the request is coming from - without storing session state on the server itself. Think of it as a signed hall pass.

---

> **Study tip:** Don't just memorise these definitions - try explaining each one to yourself using an analogy. If you can explain Accessibility Service as *"a pair of eyes that watches other apps on my behalf"*, you'll sound far more natural than reciting a dictionary definition.

---

*Last updated: June 2026 | Project: Aegis: Digital Shield | Developer: Yahya*
