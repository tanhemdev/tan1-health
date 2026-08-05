# TAN-1  -  Metrics & Results

---

## pilot overview

- **duration:** 4 weeks (Feb 2025)
- **participants:** 3 patient-caregiver pairs
- **total readings processed:** 24,192
- **total alerts generated:** 847
- **total escalations:** 100

---

## core metrics

| metric | target | actual | status |
|--------|--------|--------|--------|
| risk classification accuracy | >=90% | **92.4%** | hit |
| false urgent alert rate | <5% | **3.1%** | hit |
| median caregiver ack time | <10 min | **7.2 min** | hit |
| escalation trigger rate | <15% | **11.8%** | hit |
| time-in-range improvement | +8% over 90 days | **+6.2% at 4 weeks** | trending |
| caregiver anxiety reduction | -30% over 30 days | **-34%** | hit |
| caregiver DAU | 70% | **78%** | hit |

---

## model performance

| tier | precision | recall | f1 | support |
|------|-----------|--------|----|----------|
| low | 0.89 | 0.96 | 0.92 | 187 |
| normal | 0.95 | 0.93 | 0.94 | 612 |
| elevated | 0.88 | 0.86 | 0.87 | 198 |
| urgent | 0.91 | 0.96 | 0.93 | 47 |
| **weighted avg** | **0.92** | **0.92** | **0.92** | **1044** |

recall on low and urgent deliberately high  -  false negatives on dangerous readings are unacceptable.

---

## alert breakdown

| category | count | % of total |
|----------|-------|------------|
| normal (no alert) | 22,891 | 94.6% |
| elevated alerts | 712 | 2.9% |
| urgent alerts | 135 | 0.6% |
| low alerts | 454 | 1.9% |

average alerts per day per patient: ~10.1
after meal tagging (v1.1): ~7.3 (27% reduction in false post-meal alerts)

---

## caregiver behavior changes

| metric | before TAN-1 | after 4 weeks | change |
|--------|-------------|---------------|--------|
| daily Dexcom Follow checks | 12.4 avg | 4.1 avg | -67% |
| midnight checks | 2.1/night | 0.3/night | -86% |
| anxiety score (1-10) | 7.8 avg | 5.1 avg | -34% |
| "felt confident glucose was being monitored" | 3/10 | 8.2/10 | +173% |

---

## qualitative feedback

> "I actually slept through the night for the first time in three years. I woke up to a 'good morning, overnight was uneventful' notification and I cried."  -  P3

> "The acknowledge button changed our family dynamics. My sister and I used to panic-call each other. Now one of us taps 'I'm on it' and the other one knows."  -  P11

> "I still check Follow sometimes out of habit. But I'm checking 4 times a day instead of 20. That's a different life."  -  P7

---

## what's next

1. **daily digest**  -  morning summary of overnight readings + upcoming risk patterns
2. **multi-patient support**  -  for home health aides managing 3-5 patients
3. **longer pilot**  -  90-day study to validate time-in-range improvement target
4. **model v2**  -  incorporate insulin dosing data (with consent) for better predictions
