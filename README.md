# TAN-1 Health Ecosystem

**a glucose risk prediction system that turns CGM noise into plain-english caregiver alerts - and a community where people living with diabetes find each other.**

---

<p align="center">
  <img <img width="1280" height="1024" alt="36db6bbc-88b7-4163-9121-3cd88db8dd3c 2" src="https://github.com/user-attachments/assets/79b414c3-63c6-4375-80bb-255398d46278" />
  Tanisha and I as kids" 
  <img <img width="3120" height="4160" alt="IMG_5377" src="https://github.com/user-attachments/assets/674f9c51-abbe-4698-9d4b-063dc818cf94" />
Tanya and Tanisha today" 
</p>

## Why I built this

My twin sister Tanisha was diagnosed with Type 1 diabetes. I remember the exact moment it changed everything - not just for her, but for everyone around her. My family and I showed up in every way we knew how, but there was this quiet truth none of us could get around: no matter how present we were, we would never fully understand what she was feeling. The 3am lows. The mental math before every meal. Changing a CGM sensor every 13 days and bracing for the sting each time.

And I couldn't help but wonder: **if the people closest to Tanisha - who love her more than anything - still can't fully understand what she's going through, what does it feel like for the 40 million other Americans living with diabetes who don't even have that support?**

That sense of isolation - of living with something that the people closest to you can't truly feel - is what led me to TAN-1. I didn't want to build another glucose tracker. I wanted to build the thing I wished existed for Tanisha: a place where people living with the same condition could find each other. Where a 14-year-old could learn from a 67-year-old that pressing an ice cube to your skin before inserting the CGM sensor makes it hurt less. Where someone newly diagnosed could ask a question at 2am and actually get an answer from someone who gets it.

I interviewed over **80 people** - pre-teens, college students, parents, senior citizens - to understand what they actually needed. Not what a product roadmap said they needed. What I heard again and again was that the hardest part wasn't the disease itself. It was feeling like no one else understood.

The technical side: TAN-1 ingests continuous glucose monitor data, runs it through a risk classification model (92% accuracy across 10K+ readings), and sends real-time alerts to caregivers in plain english. Not "glucose: 247 mg/dL" - instead: "Tanisha's blood sugar has been high for 2 hours. This might need attention. [Acknowledge]"

Presented at CalHacks Health and Stanford TreeHacks to 200+ attendees combined.

*To Tanisha - this journey is not even 1% of what you go through every single day. But if it helps even one person feel a little less alone, I think it's worth everything.*

---

## How it works

1. **CGM data lands** - Dexcom G6/G7 API pushes 5-minute interval readings to our ingestion service
2. **feature extraction** - we compute rolling averages, rate of change, time-in-range windows, and meal proximity flags
3. **risk classification** - scikit-learn gradient boosted classifier bins readings into low / normal / elevated / urgent
4. **alert generation** - urgent and elevated readings trigger plain-english alerts through the notification engine
5. **caregiver delivery** - push notification hits the caregiver's phone with one-tap acknowledge
6. **ack loop closes** - if no acknowledgment in 15 min, escalation kicks in (second caregiver, then phone call via Twilio)

---

## tech stack

| layer | tech | why |
|-------|------|-----|
| ml pipeline | Python, scikit-learn, pandas | fast iteration, good enough for tabular medical data, interpretable |
| api | FastAPI | async support, auto-generated docs, type hints |
| mobile app | React Native (Expo) | cross-platform, my caregiver testers use both iOS and Android |
| database | PostgreSQL + TimescaleDB | time-series glucose data needs hypertable performance |
| notifications | Firebase Cloud Messaging + Twilio | push + SMS/call fallback for escalation |
| monitoring | Datadog | need to know if alert pipeline goes down at 3am |
| auth | Firebase Auth | simple, handles the family-sharing model well |

---

## docs

- [Product Requirements Document](docs/PRD.md) - the why and what
- [User Research](docs/USER_RESEARCH.md) - interviews with diabetics and caregivers
- [Architecture](docs/ARCHITECTURE.md) - system design and data flow
- [Sprint Log](docs/SPRINT_LOG.md) - what actually happened, sprint by sprint
- [Metrics](docs/METRICS.md) - post-launch numbers

---

## license

MIT - use it, fork it, adapt it for your family. if you do, let me know how it goes.

built by [tanya hemdev](https://github.com/tanhemdev)
