# Broadcast_Director_AI
AI Assistant for Broadcast Director
# Broadcast Director AI System

## Overview

An intelligent camera direction system that helps livestream directors know which camera to cut to next, synced to music/beat, with real-time YouTube metadata generation. Designed for church livestreams but applicable to any live broadcast scenario.

**Core Problem:** Director gets stuck on the same camera, doesn't know what's coming next, struggles to stay on beat.

**Solution:** AI watches the live stream, predicts next camera cut 30-90 seconds in advance with countdown timer, and generates YouTube titles/descriptions in real-time.

---

## Features

### Feature 1: Camera Prediction & Countdown Timer

**What it does:**
- Monitors all camera angles from ATEM simultaneously
- Predicts which camera should be next
- Displays countdown on secondary monitor: `NEXT: Cam 2 | 3 — 2 — 1 — CUT`
- Syncs cuts to beat/music timing

**How it works:**
1. Pre-service setup: Director inputs song order + who leads + stage position for each
2. During service: AI detects person's position on stage using position detection
3. Song list + detected position = camera prediction
4. Beat detection determines when to cut (on rhythm)
5. Countdown timer alerts director 30-60 seconds before cut

**Input data:**
- Live ATEM feed (all cameras)
- Song schedule for the day
- Stage position mapping (e.g., "John = stage left = Camera 1")
- Audio feed (for beat detection)

**Output:**
```
NEXT: Camera 2 (Amy - Center)
Stage: Center
━━━━━━━━━━━━━━━━
  3 — 2 — 1 — CUT
```

### Feature 2: YouTube Title & Description Generation

**What it does:**
- Transcribes sermon audio in real-time or post-service
- Extracts text from slides (Scripture references, key points)
- Generates professional YouTube title and description
- Director reviews and approves with one click

**How it works:**
1. Audio from ATEM → speech-to-text (Whisper)
2. Video frames with slides → OCR extraction
3. Combine transcript + slide text → Claude API
4. Return formatted title + description ready for YouTube

**Example output:**
```
Title: Pastor Steve Speaks on Faith and Perseverance
Description: Join us as Pastor Steve explores what it means to have 
faith in uncertain times. Scripture: Hebrews 11:1-6. Trust in God 
when trials come.
```

**Cost per service:** ~$1-2 (Whisper + Claude API)

---

## Pre-Service Setup (5 Minutes)

Director fills out a simple form before each service:

```
SUNDAY SERVICE SETUP

Song 1: Opening
├─ Leader: John
└─ Position: Stage Left

Song 2: Worship Set  
├─ Leader: Amy
└─ Position: Stage Center

Song 3: Worship Continues
├─ Leader: Band
└─ Position: Full Stage

Song 4: Sermon
├─ Leader: Pastor Steve
└─ Position: Center
```

This is the only input needed per service. Everything else is automated.

---

## Auto-Training from YouTube

Instead of manually annotating past services, the system trains itself:

### Process

1. **You provide:** YouTube channel link (e.g., `youtube.com/@yourchurch`)
2. **AI downloads:** Last 50 livestreams (~1 year of services)
3. **AI analyzes:** 
   - Detects camera cuts automatically (scene change detection)
   - Identifies transitions frame-by-frame
   - Marks each cut with timestamp
4. **Optional:** Provide past song schedule (spreadsheet)
   - Helps AI understand *why* cuts were made
   - Improves accuracy from 75% → 85%+
5. **Model trains:** On 50+ services × 20+ cuts each = 1000+ training examples
6. **Result:** AI learns your specific director patterns

### Timeline
- Day 1: YouTube link + optional schedule provided
- Day 2-3: Download & analysis (runs overnight)
- Day 4: Model trained and ready
- Sunday: Live deployment

### Accuracy
- Without schedule: 75%+ accuracy
- With schedule: 85-90% accuracy

---

## Architecture

### System Components

```
Director's Computer (Mac/PC)
├─ Frontend (Electron app)
│  ├─ Pre-service setup form
│  ├─ Live director monitor
│  │  ├─ Camera countdown timer
│  │  ├─ Next camera suggestion
│  │  └─ Manual override controls
│  └─ Post-service title review
│
├─ Backend (Python)
│  ├─ ATEM API connector
│  │  └─ Pulls all camera feeds real-time
│  ├─ Position detector
│  │  └─ MediaPipe Pose (detect stage position)
│  ├─ Song schedule engine
│  │  └─ Matches current song to preset data
│  ├─ Beat detector
│  │  └─ Audio analysis for rhythm sync
│  ├─ Prediction model
│  │  └─ LSTM/Transformer on trained data
│  ├─ Speech-to-text engine
│  │  └─ Whisper (OpenAI)
│  ├─ Slide OCR
│  │  └─ Extract text from video frames
│  └─ Title generation
│     └─ Claude API calls
│
└─ Local Storage
   ├─ Service configurations
   ├─ Training data
   └─ Past transcripts
```

### Data Flow (Real-Time)

```
ATEM Feed → Position Detector → Stage Position
                                    ↓
Song Schedule + Position → Prediction Model → Next Camera + Confidence
                                    ↓
ATEM Audio → Beat Detector → Timing for Cut
                                    ↓
Director Monitor Display ← Combine all above
                                    ↓
Director Sees: "NEXT: Cam 2 | 3 — 2 — 1 — CUT"
                                    ↓
Director Confirms/Overrides → ATEM Switches Camera
```

### Data Flow (Title Generation)

```
ATEM Audio → Whisper (Speech-to-Text) → Sermon Transcript
                                    ↓
ATEM Video → OCR → Slide Text + Scripture References
                                    ↓
Combine Transcript + Slides → Claude API
                                    ↓
Generated Title + Description
                                    ↓
Director Review → YouTube Upload
```

---

## Technical Stack

### Frontend
- **Electron** — Desktop app (Mac + Windows)
- **React + TypeScript** — UI components
- **Vite** — Build tool

### Backend
- **Python 3.10+** — Core engine
- **FastAPI** — Local API server
- **asyncio** — Real-time processing

### AI/ML
- **MediaPipe Pose** — Body/position detection
- **OpenAI Whisper** — Speech-to-text
- **PyTorch** — Model training & inference
- **Scikit-learn** — Feature extraction
- **OpenCV** — Video analysis

### APIs
- **Blackmagic ATEM API** — Camera switcher control
- **OpenAI API** — Claude (title generation), Whisper (transcription)
- **yt-dlp** — YouTube video downloading

### Storage
- **SQLite** — Local database (configs, schedules)
- **File system** — Training data, service recordings

---

## Implementation Timeline

### Phase 1: Core Camera Prediction (Week 1-2)

**Day 1-2: Setup & Data Prep**
- ATEM SDK integration
- YouTube download script
- Auto scene detection from past streams

**Day 3-4: Position Detection**
- MediaPipe Pose integration
- Stage zone mapping
- Frame extraction pipeline

**Day 5-7: Model Training**
- Training data preparation (50 services)
- LSTM model training
- Accuracy testing

### Phase 2: Director UI & Countdown (Week 2-3)

**Day 8-10: Frontend Setup**
- Electron app skeleton
- Pre-service setup form
- Director monitor display

**Day 11-13: Real-Time Integration**
- ATEM feed connection
- Countdown timer
- Override controls

**Day 14-15: Testing & Refinement**
- Test on past services
- Calibrate timing
- Manual cut override testing

### Phase 3: YouTube Title Generation (Week 3-4)

**Day 16-18: Audio & OCR**
- Whisper integration
- Slide text OCR
- Data cleaning

**Day 19-21: Title Generation**
- Claude API integration
- Prompt engineering
- Title + description formatting

**Day 22-23: UI Integration**
- Review interface
- Edit/regenerate options
- Auto-post capability

### Phase 4: Live Testing at Church (Week 4+)

**Sunday 1:** Test camera prediction only
**Sunday 2:** Add countdown timer
**Sunday 3:** Full system with title generation
**Sunday 4+:** Live deployment & refinement

---

## Stage Position Mapping (Example)

Maps where people stand to which camera covers that area:

```
Stage Layout:

Stage Left (Left Third)  |  Stage Center (Middle Third)  |  Stage Right (Right Third)
Camera 1 Focus          |  Camera 2 Focus              |  Camera 3 Focus
├─ John                 |  ├─ Amy                      |  ├─ Drummer
├─ Worship Leader 1     |  ├─ Pastor Steve             |  ├─ Bass Player
└─ (zoom available)     |  ├─ Center microphone        |  └─ (zoom available)
                        |  └─ Main stage position      |
```

**Song Schedule with Positions:**

```
Song 1: Opening (John) → Position: Stage Left → Camera: 1
Song 2: Worship (Amy) → Position: Stage Center → Camera: 2
Song 3: Band Join → Position: Full Stage → Cameras: 1 + 2 + 3 (wide)
Song 4: Sermon (Pastor Steve) → Position: Stage Center → Camera: 2
```

---

## Pre-Service Setup Form (UI Mock)

```
╔════════════════════════════════════════════╗
║   CHURCH BROADCAST SETUP                   ║
╠════════════════════════════════════════════╣
║                                            ║
║ SONG 1: Opening                            ║
║ ├─ Leader: [John dropdown]                 ║
║ └─ Position: [Stage Left v]                ║
║                                            ║
║ SONG 2: Worship Set                        ║
║ ├─ Leader: [Amy dropdown]                  ║
║ └─ Position: [Stage Center v]              ║
║                                            ║
║ SONG 3: Worship Continues                  ║
║ ├─ Leader: [Band dropdown]                 ║
║ └─ Position: [Full Stage v]                ║
║                                            ║
║ SONG 4: Sermon                             ║
║ ├─ Leader: [Pastor Steve dropdown]         ║
║ └─ Position: [Stage Center v]              ║
║                                            ║
║ Optional: Song Schedule Spreadsheet        ║
║ └─ [Upload or Skip]                        ║
║                                            ║
║              [SAVE & START BROADCAST]      ║
╚════════════════════════════════════════════╝
```

---

## Director Monitor During Service

```
╔════════════════════════════════════════════╗
║   BROADCAST DIRECTOR AI - LIVE             ║
╠════════════════════════════════════════════╣
║                                            ║
║ Current Song: Worship Set (Amy)            ║
║ Current Position: Stage Center             ║
║ Current Camera: Camera 2                   ║
║                                            ║
║ ─────────────────────────────────────────  ║
║                                            ║
║ NEXT: Camera 3 (Band - Stage Right)        ║
║ Confidence: 87%                            ║
║                                            ║
║         3 — 2 — 1 — CUT                    ║
║                                            ║
║ ─────────────────────────────────────────  ║
║                                            ║
║ [Manual Override] [Postpone] [Skip Suggestion] ║
║                                            ║
╚════════════════════════════════════════════╝
```

---

## YouTube Title Generation UI

```
╔════════════════════════════════════════════╗
║   YOUTUBE METADATA GENERATION - POST LIVE  ║
╠════════════════════════════════════════════╣
║                                            ║
║ Title (DRAFT):                             ║
║ Pastor Steve Speaks on Faith and           ║
║ Perseverance                               ║
║                                            ║
║ Description (DRAFT):                       ║
║ Join us as Pastor Steve explores what it   ║
║ means to have faith in uncertain times.    ║
║                                            ║
║ Scripture: Hebrews 11:1-6                  ║
║ Topics: Faith, Trust, Perseverance         ║
║                                            ║
║ ─────────────────────────────────────────  ║
║                                            ║
║ [✓ Use This] [✎ Edit] [🔄 Regenerate]     ║
║ [Auto-Post to YouTube]                     ║
║                                            ║
╚════════════════════════════════════════════╝
```

---

## Cost Analysis

### Development Cost (For You)
- **Your time:** ~80-100 hours
- **External design:** $0 (you're handling it)
- **APIs:** Free tier initially, paid once tested

### Per-Service Operating Cost
- **Whisper transcription:** $0.90
- **Claude API calls:** $0.20-0.50
- **ATEM API:** Free
- **YouTube API:** Free
- **Total per service:** ~$1.20

*Note: This is only if you auto-transcribe/generate. Can be $0 if manual.*

### Licensing
- **Blackmagic SDK:** Free
- **OpenAI API:** Pay-as-you-go
- **MediaPipe:** Free
- **PyTorch:** Free
- **Electron:** Free
- **Code signing (Mac):** $99/year

### Total Startup Cost
- ~$100-200 for initial setup
- ~$50-100/year for code signing & misc

---

## Why This Works

### For Your Church (MVP)
✅ Solves real problem (you getting stuck on same camera)
✅ Syncs to music/beat (professional timing)
✅ Minimal setup (5 minutes per service)
✅ Runs in background (no disruption)
✅ YouTube titles automated (saves time)
✅ All data local (no privacy concerns)
✅ Can still manually override (not fully automated)

### For Sellable Product
✅ No direct competitors (blue ocean)
✅ Clear market (thousands of churches streaming)
✅ Auto-training from YouTube (easy onboarding)
✅ Recurring revenue model (subscription)
✅ Proven by your own use case (you're the expert)
✅ Natural feature expansion (more cameras, OBS support, Tricaster, etc.)
✅ Scalable infrastructure (runs locally, optional cloud backup)

---

## Realistic Limitations

⚠️ **Position detection accuracy** 
- Works well with good camera angles
- Fails if people move off-stage or behind objects
- Requires consistent stage layout

⚠️ **Transcription latency**
- Real-time: 10-30 seconds behind live audio
- Post-service: Near-instant (2-3 min turnaround)

⚠️ **Slide OCR quality**
- Works best with plain text slides
- Struggles with fancy design/embedded graphics
- Requires decent resolution video feed

⚠️ **Pattern consistency**
- Works best when services follow similar structure
- Guest speakers or unusual formats break predictions
- Director can always override

⚠️ **API costs** 
- Adds ~$1.20/service if using Whisper + Claude
- Can be eliminated if manual transcription acceptable

⚠️ **ATEM-only initially**
- Supports ATEM switchers only
- Tricaster, vMix, OBS would need separate integration

---

## Product Positioning (Future)

### Problem Statement
Church livestream directors struggle with:
- Not knowing which camera to cut to next
- Missing musical beats for timing
- Manually creating YouTube titles/descriptions
- Inconsistent broadcast quality

### Solution
Intelligent director assistant that:
- Predicts next camera cut 30-90 seconds in advance
- Syncs cuts to music/beat
- Auto-generates YouTube metadata
- Learns from your past broadcasts

### Target Market
- **Primary:** Churches with 300+ attendance doing weekly livestreams
- **Secondary:** Small studios, corporate events, school assemblies
- **Tertiary:** Theater productions, community streams

### Market Size
- ~300,000 churches in US
- ~50,000 actively livestreaming
- ~10,000 willing to pay for tools
- Average budget: $2-10k/year for broadcast tech

### Pricing Model (Subscription)
- **Solo church:** $20-30/month (1 ATEM switcher)
- **Multi-campus:** $50-100/month (unlimited switchers)
- **Plus title generation:** +$10/month

### Revenue Potential
- 500 churches at $25/month = $150k/year
- 2000 churches at $25/month = $600k/year
- Room to scale to 5000+ eventually

---

## Getting Started (Next Steps)

### Week 1: Foundation
- [ ] Gather YouTube channel link
- [ ] Collect past song schedules (if available)
- [ ] Download 5-10 recent livestreams
- [ ] Set up Python environment on Mac
- [ ] Install dependencies (yt-dlp, opencv, torch, etc.)

### Week 2: Training
- [ ] Build scene detection script
- [ ] Auto-analyze downloaded streams
- [ ] Generate training dataset
- [ ] Build position detector
- [ ] Train initial model

### Week 3: UI & Real-Time
- [ ] Build Electron app skeleton
- [ ] Create pre-service setup form
- [ ] Connect to ATEM API
- [ ] Build director monitor display
- [ ] Implement countdown timer

### Week 4: Integration & Testing
- [ ] Test camera prediction on past services
- [ ] Integrate with live ATEM feed
- [ ] Add manual override controls
- [ ] Test countdown timing
- [ ] Refine model accuracy

### Week 5: Polish & Deploy
- [ ] Add YouTube title generation
- [ ] Build post-service review UI
- [ ] Add OCR for slides
- [ ] Final testing
- [ ] Deploy to church

---

## Key Questions Answered

**Q: Do I need expensive hardware?**
A: No. Runs on your existing Mac. ATEM connection is only requirement.

**Q: How long to build?**
A: MVP for your church: 4-5 weeks. Product-ready: 2-3 months.

**Q: Can it work with my current ATEM setup?**
A: Yes. Blackmagic ATEM API supports all models (Mini, 2M/E, 4M/E, etc.)

**Q: What if my stage setup changes?**
A: Just update the pre-service form. AI adapts immediately.

**Q: Can I use it without YouTube training?**
A: Yes, but less accurate. Recommend at least 10-15 past services.

**Q: Can this be sold to other churches?**
A: Absolutely. That's the plan. They paste their YouTube link, system self-trains.

**Q: What if something goes wrong during service?**
A: You maintain full manual control. Suggestions are just that—suggestions.

**Q: How much does it cost per service?**
A: ~$1.20 if auto-generating titles. Free if you skip title generation.

---

## Success Metrics

### For Your Church (MVP)
- ✓ Director no longer gets stuck on same camera
- ✓ Cuts sync to musical beats
- ✓ YouTube titles generated in <5 minutes post-service
- ✓ Camera suggestions accurate 80%+ of the time
- ✓ System runs reliably for 4+ consecutive services

### For Product
- ✓ 5-10 beta churches using system
- ✓ >80% camera prediction accuracy
- ✓ <30 second setup per service
- ✓ <$2 cost per service
- ✓ Customer satisfaction >4.5/5 stars
- ✓ Repeatable onboarding process

---

## Dependencies & Libraries

```python
# Core
python>=3.10
torch>=2.0
opencv-python>=4.8
numpy>=1.24
pandas>=2.0

# ATEM
blackmagic-decklink  # ATEM SDK

# AI/ML
mediapipe>=0.10
librosa>=0.10
scikit-learn>=1.3
pytorchvideo>=0.1

# APIs
openai>=1.0  # Claude + Whisper
yt-dlp>=2024.0
requests>=2.31

# Web/UI
fastapi>=0.104
uvicorn>=0.24
pydantic>=2.0

# Frontend (Node)
electron>=27
react>=18
typescript>=5.3
```

---

## Summary

This is a **practical, buildable MVP** that solves your real problem (not knowing what camera to cut to next) while laying groundwork for a **scalable product** (selling to other churches).

**Start with the MVP:** 
- Build it for your church first
- Test every Sunday for a month
- Refine based on real use

**Then evolve to product:**
- Package it properly (Electron app)
- Add documentation
- Find beta churches
- Sell it

**Timeline:** MVP ready in 4-5 weeks, product-ready in 3-4 months.

---

*Last updated: May 31, 2026*
