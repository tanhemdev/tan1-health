# TAN-1  -  System Architecture

---

## high-level overview

TAN-1 is an event-driven pipeline. CGM readings flow in through the Dexcom webhook, get enriched with computed features, hit the risk classifier, and trigger alerts when thresholds are crossed.

```
Dexcom G6/G7 --> Ingestion --> Feature Engine --> Risk Classifier
                                                       |
                                                  Alert Generator
                                                       |
                                                Notification Service
                                                  /           \
                                                FCM          Twilio
                                              (push)       (SMS/call)
                                                       |
                                                React Native App
                                            Patient + Caregiver views
```

---

## data flow: reading ingestion -> alert delivery

```
Dexcom webhook POST
    |
/api/v1/webhooks/dexcom
    |
    +-- validate signature + parse payload
    +-- store raw reading in glucose_readings hypertable
    |
Feature Engine
    +-- query last 120 min of readings
    +-- compute: rolling averages, rate of change, time-in-range,
        meal proximity, hour of day, missing data flag
    |
Risk Classifier (sklearn GradientBoostingClassifier)
    +-- predict: low | normal | elevated | urgent
    +-- confidence score (0-1)
    +-- elevated + confidence > 0.7 -> alert
    +-- urgent + confidence > 0.5 -> alert (lower threshold for safety)
    |
Alert Generator
    +-- plain-english template: "Dad's blood sugar has been high (267) for 2 hours"
    +-- attach metadata
    |
Notification Service
    +-- FCM push to primary caregiver
    +-- 15-min escalation timer (Redis)
    +-- on ack -> cancel timer, notify others
    +-- on expiry -> next caregiver -> Twilio voice call
```

---

## ML model

### why gradient boosting and not deep learning

1. **tabular data** - glucose readings + features = numbers. GBM is king for tabular.
2. **interpretability** - health app needs explainable alerts.
3. **dataset size** - 10,400 labeled readings. enough for GBM, not for deep learning.
4. **latency** - sub-millisecond inference on 11 features.
5. **simplicity** - 2.3MB pickle file. joblib.load() and go.

### features (11 total)

| feature | type | description |
|---------|------|-------------|
| glucose_current | float | current reading in mg/dL |
| rolling_avg_30m | float | mean of last 6 readings |
| rolling_avg_60m | float | mean of last 12 readings |
| rolling_avg_120m | float | mean of last 24 readings |
| rate_of_change | float | glucose delta per minute over last 15 min |
| time_in_range_3h | float | % of last 3h readings in 70-180 mg/dL |
| time_above_250_3h | float | % of last 3h above 250 |
| time_below_70_3h | float | % of last 3h below 70 |
| meal_proximity | bool | within configured meal window |
| hour_of_day | int | 0-23 |
| readings_missing | bool | gap > 15 min in recent data |

### performance

| tier | precision | recall | f1 | support |
|------|-----------|--------|----|---------|
| low | 0.89 | 0.96 | 0.92 | 187 |
| normal | 0.95 | 0.93 | 0.94 | 612 |
| elevated | 0.88 | 0.86 | 0.87 | 198 |
| urgent | 0.91 | 0.96 | 0.93 | 47 |
| **weighted avg** | **0.92** | **0.92** | **0.92** | **1044** |

recall on low and urgent is intentionally high  -  we'd rather over-alert than miss a dangerous reading.

---

## database schema

PostgreSQL 15 with TimescaleDB extension.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('patient', 'caregiver')),
    fcm_token TEXT,
    phone TEXT,
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE care_circles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES users(id),
    caregiver_id UUID REFERENCES users(id),
    priority INT NOT NULL DEFAULT 1,
    created_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(patient_id, caregiver_id)
);

CREATE TABLE glucose_readings (
    time TIMESTAMPTZ NOT NULL,
    patient_id UUID NOT NULL REFERENCES users(id),
    glucose_mg_dl FLOAT NOT NULL,
    trend_arrow TEXT,
    source TEXT DEFAULT 'dexcom',
    risk_tier TEXT,
    risk_confidence FLOAT,
    features JSONB,
    PRIMARY KEY (time, patient_id)
);
SELECT create_hypertable('glucose_readings', 'time');

CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID NOT NULL REFERENCES users(id),
    reading_time TIMESTAMPTZ NOT NULL,
    risk_tier TEXT NOT NULL,
    message TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    acknowledged_at TIMESTAMPTZ,
    acknowledged_by UUID REFERENCES users(id),
    escalated BOOLEAN DEFAULT false,
    escalation_level INT DEFAULT 0
);
```

90 days of readings in the hot partition (~26K rows per patient). not a scale problem.

---

## API design decisions

- **REST over GraphQL** - one client, 4-5 queries. REST is simpler.
- **WebSocket for dashboard** - real-time glucose display via Socket.IO.
- **FCM + Twilio** - FCM for push (free, fast), Twilio for SMS/voice escalation.
- **Firebase Auth** - custom claims for patient/caregiver roles. care circle invites use a 6-digit code.
