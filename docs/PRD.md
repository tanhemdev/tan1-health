# TAN-1 Health Ecosystem  -  Product Requirements Document

**Author:** Tanya Hemdev
**Last updated:** March 2025
**Status:** Shipped (v1.2)

---

## problem statement

A dad got diagnosed with Type 2 diabetes last year. his endocrinologist put him on a Dexcom G6, and suddenly he was getting glucose readings every 5 minutes  -  288 data points per day  -  with zero context on what any of them meant.

he'd see "247 mg/dL" and text me: *"is this bad?"* he'd wake up at 3am with a low alarm blaring and not know if he should eat glucose tabs or just go back to sleep. my mom, who's his primary caregiver, was checking the Dexcom Follow app obsessively but couldn't tell the difference between a post-meal spike (normal) and a sustained high that actually needed intervention.

the existing tools (Dexcom Follow, Sugarmate, etc.) show you the number and a trend arrow. that's it. no interpretation, no "here's what you should do," no way for a caregiver to say "got it, i'm on it" so other family members aren't also panicking.

the gap: **there's no system that translates raw glucose data into actionable, plain-english guidance and closes the communication loop between patient and caregiver.**

---

## target users

### primary
- **caregivers of diabetics**  -  family members (spouse, parent, adult child) who are responsible for helping manage someone else's diabetes but don't have medical training
- **Type 1 and Type 2 diabetics** who want better self-monitoring without staring at graphs all day

### secondary
- **parents of children with Type 1**  -  highest anxiety user segment, most likely to over-check
- **home health aides** managing multiple patients

### explicitly not targeting (for now)
- clinical teams / hospital settings (regulatory nightmare, different alert thresholds, EHR integration)
- people without CGMs (no continuous data = model doesn't work)

---

## user personas

### Maya, 52  -  "The Worried Spouse"
Maya's husband Raj was diagnosed with T2D 8 months ago. She checks Dexcom Follow 15-20 times per day. She doesn't understand the difference between time-in-range and A1C. When she sees a high reading, she texts Raj, their daughter, and sometimes calls the doctor's office  -  even when it's a normal post-meal spike. She's exhausted and anxious.

**needs:** confidence that she'll be alerted when it actually matters. fewer false alarms. a way to say "i saw this, i'm handling it" so her daughter stops worrying too.

### Kevin, 23  -  "The Overwhelmed Patient"
Kevin has had T1D since age 11. He's in college, eating dining hall food, and his glucose control has slipped. His A1C went from 6.8 to 7.9. He gets Dexcom alerts but silences them because they go off during class. His mom calls him every morning to ask about his numbers.

**needs:** a system that filters out the noise and only alerts him (and his mom) when something genuinely needs attention. wants his mom to stop calling.

### Priya, 34  -  "The Long-Distance Daughter"
Priya lives in SF. Her dad is in Fremont, managing T2D alone since her mom passed. She checks Dexcom Follow before bed and first thing in the morning. She's terrified of overnight lows. She hired a part-time aide but has no way to know if the aide is actually responding to alerts.

**needs:** escalation. if dad's glucose drops and the aide doesn't respond, Priya needs to know. wants an audit trail of who acknowledged what.

---

## success metrics

| metric | target | rationale |
|--------|--------|-----------|
| risk classification accuracy | >=90% | below this, caregivers lose trust in the system |
| false urgent alert rate | <5% | alert fatigue is the #1 reason people abandon glucose apps |
| median caregiver acknowledgment time | <10 min | measures whether alerts are actually reaching people |
| escalation trigger rate | <15% of alerts | if we're escalating too much, primary caregivers aren't engaged |
| time-in-range improvement (patient) | +8% over 90 days | clinical outcome  -  are we actually helping? |
| caregiver anxiety score (custom survey) | -30% over 30 days | qualitative but important |
| DAU (caregivers) | 70% of registered | caregivers should check in daily |

**current actuals (as of v1.2):**
- risk classification accuracy: **92.4%**
- false urgent rate: **3.1%**
- median ack time: **7.2 minutes**
- escalation rate: **11.8%**

---

## feature requirements

### P0  -  must ship in v1

| feature | description | status |
|---------|-------------|--------|
| CGM data ingestion | receive Dexcom G6/G7 readings via API webhook, store in TimescaleDB | shipped |
| risk classification | classify each reading into low/normal/elevated/urgent using trained model | shipped |
| plain-english alerts | convert risk classification into human-readable caregiver notification | shipped |
| caregiver acknowledgment | one-tap ack on push notification, recorded in alert history | shipped |
| escalation | if no ack within 15 min, notify next caregiver in chain | shipped |
| patient dashboard | current reading, 24h trend, time-in-range, risk distribution | shipped |
| caregiver alert feed | chronological list of alerts with ack status | shipped |

### P1  -  ship in v1.x

| feature | description | status |
|---------|-------------|--------|
| meal tagging | patient can tag meals to reduce false post-meal spike alerts | shipped (v1.1) |
| custom alert thresholds | caregiver can adjust what counts as "elevated" for their patient | shipped (v1.2) |
| daily digest | morning summary of overnight readings + risk events | in progress |
| multi-patient support | for aides managing multiple patients | spec'd |

### P2  -  future

| feature | description | status |
|---------|-------------|--------|
| A1C estimation | predict A1C from 90-day CGM data | researching |
| medication reminders | tie alerts to insulin dosing schedule | spec'd |
| clinician portal | read-only dashboard for endocrinologist check-ins | not started |
| Libre integration | support Abbott FreeStyle Libre in addition to Dexcom | not started |

---

## out of scope

- **medical advice.** TAN-1 does not tell you what to do. it tells you something might need attention. we're very deliberate about this  -  the alert says "this might need attention," not "take 15g of fast-acting carbs." we're not a medical device and we're not trying to be.
- **insulin dosing calculations.** liability and regulatory risk too high. this is a monitoring + communication tool.
- **EHR/EMR integration.** would love to eventually, but HIPAA BAA + HL7 FHIR integration is a 6-month project minimum and we're a team of one.
- **Android Wear / Apple Watch companion app.** would be cool, low priority. push notifications already hit the watch.

---

## risks and mitigations

| risk | severity | likelihood | mitigation |
|------|----------|------------|------------|
| false negative on urgent low | critical | low (recall for urgent tier: 96.2%) | aggressive threshold for lows  -  we'd rather over-alert on lows than miss one |
| alert fatigue leads to ignored notifications | high | medium | smart batching: if glucose is consistently elevated, we send one alert with context, not 12 alerts in an hour |
| Dexcom API downtime | medium | medium | 30-min data buffer; if no new reading in 35 min, send "we haven't received data" alert |
| user stores real patient data | medium | high | all data encrypted at rest, we never display full names in alerts, PII scrubbed from logs |
| model drift as user population changes | medium | low (for now) | monthly accuracy audit against holdout set; retrain trigger if accuracy drops below 89% |
| scope creep toward medical device territory | high | medium | clear product principles doc; legal reviewed alert language; explicit "not medical advice" disclaimers |

---

## open questions

1. should we let patients opt out of caregiver alerts for specific time windows? (e.g., Kevin doesn't want his mom alerted during class). leaning yes but need to think through the liability angle.
2. what's the right escalation timeout  -  15 min feels right for urgent, but elevated readings might need a longer window (30 min?). running an experiment.
3. should we build a web dashboard for caregivers who don't want another app? initial research says no  -  push notifications are the killer feature, and a web app doesn't do those well.
