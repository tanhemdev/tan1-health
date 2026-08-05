# TAN-1  -  Sprint Log

---

## sprint 1: data pipeline (weeks 1-2)

**goal:** get CGM data flowing from Dexcom into our database.

**what happened:**
- set up FastAPI project structure, PostgreSQL + TimescaleDB
- built Dexcom webhook receiver
- created glucose_readings hypertable
- got first real readings flowing from dad's Dexcom G6

**blockers:**
- Dexcom OAuth token refresh was silently failing. timezone issue in token expiry check.

---

## sprint 2: risk classification (weeks 3-4)

**goal:** build and train the ML model.

**what happened:**
- collected 10,437 labeled readings from 3 participants + OhioT1DM dataset
- engineered 11 features (rolling averages, rate of change, time-in-range)
- tried logistic regression, random forest, GBM. GBM won at 92.4% weighted accuracy
- upweighted low and urgent classes 3x for safety
- deployed as pickle file in FastAPI process

**what i learned:**
- meal proximity flag was the single biggest accuracy improvement (+4%)
- chose sklearn over TensorFlow. tabular data, small dataset, interpretability.

---

## sprint 3: alert system (weeks 5-6)

**goal:** turn risk classifications into plain-english alerts.

**what happened:**
- built alert template engine with 12 templates
- integrated Firebase Cloud Messaging
- built caregiver acknowledgment flow (one-tap from push notification)
- implemented escalation chain with Redis-backed 15-min timer

**what i learned:**
- alert language matters. rewrote from "URGENT: glucose critically high" to "Dad's blood sugar has been high for 2 hours. This might need attention." Same info, completely different emotional response.

---

## sprint 4: mobile app (weeks 7-9)

**goal:** build the React Native app.

**what happened:**
- patient dashboard: current reading, 24h sparkline, time-in-range donut
- caregiver alert feed with ack status and resolution tracking
- WebSocket integration for real-time updates
- added "resolved" notifications from user feedback

**scope cut:**
- removed daily digest (alert system was sufficient)
- removed A1C estimation (not enough data yet)

---

## sprint 5: polish + pilot (weeks 10-12)

**goal:** fix bugs, add meal tagging, run pilot.

**what happened:**
- added meal tagging (suppresses post-meal spike alerts for 90 min)
- added custom alert thresholds
- 4-week pilot with 3 patient-caregiver pairs
- fixed 23 bugs
- presented at CalHacks Health and Stanford TreeHacks

**pilot results:**
- 92.4% risk classification accuracy
- 3.1% false urgent rate (target <5%)
- 7.2 min median caregiver ack time (target <10 min)
- 11.8% escalation rate (target <15%)
- all 3 caregivers reported feeling "less anxious" by week 3
