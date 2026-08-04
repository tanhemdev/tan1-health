# TAN-1  -  User Research Synthesis

**Researcher:** Tanya Hemdev
**Dates:** Sep 2024  -  Nov 2024
**Participants:** 14 (8 caregivers, 4 T1D patients, 2 T2D patients)

---

## methodology

### phase 1: exploratory interviews (n=9)
45-minute semi-structured interviews over Zoom. recruited through the Bay Area Type 1 Parents Facebook group, the Berkeley diabetes support network, and my own family connections. compensated with $25 Amazon gift cards.

**screener criteria:**
- must be actively using a CGM (Dexcom or Libre) or be a caregiver for someone who does
- must have been managing diabetes for at least 6 months
- caregivers must be checking glucose data at least once per day

### phase 2: diary study (n=6)
5-day diary study where participants logged every time they checked glucose data, what they saw, how they felt, and what they did about it. used a simple Google Form they could fill out on their phone. 4 caregivers, 2 patients.

### phase 3: concept testing (n=8)
showed Figma prototypes of the alert system and ack flow to 5 returning participants + 3 new recruits. tested comprehension, trust, and willingness to rely on the system.

---

## key findings

### 1. caregivers check CGM apps way more than they need to  -  and they know it

every single caregiver participant described compulsive checking behavior. the median was 12 checks per day, with one parent reporting 30+.

> "I check Dexcom Follow before I check my email in the morning. I check it in the middle of meetings. I know most of the time it's fine, but the one time I don't check..."  -  P3, mother of 14yo with T1D

> "My husband told me I have a problem. He's probably right. But his mother doesn't have diabetes, so he doesn't get it."  -  P7, daughter caring for father with T2D

**implication:** the product needs to actively reduce checking behavior by building trust that alerts will catch what matters. if we succeed, DAU might actually go *down*  -  and that's a good thing.

### 2. trend arrows are useless to non-medical caregivers

Dexcom shows a trend arrow alongside the glucose number. every caregiver we talked to either ignored the arrow or misinterpreted it.

> "I know the up arrow is bad? But sometimes it goes up after he eats and that's normal? I honestly don't know when to worry about the arrow."  -  P2, wife of T2D patient

> "I had to google what the diagonal arrow meant. I still don't really get it."  -  P9, father of 8yo with T1D

**implication:** our alert language cannot assume any medical literacy. "your son's glucose has been rising for 90 minutes" is better than "glucose trending up at 2.1 mg/dL/min."

### 3. the acknowledge gap is real and causes family conflict

caregivers described scenarios where multiple family members all saw a concerning reading and either all panicked simultaneously or all assumed someone else was handling it.

> "My sister and I both have Follow. When Dad goes low, we both call him. Then we call each other. Then we call his aide. It's chaos every single time."  -  P11, long-distance caregiver

> "I found out my daughter had a low at 2am because my wife didn't tell me she'd already handled it. I was up for an hour worrying before I finally texted her."  -  P9

**implication:** the ack loop isn't a nice-to-have  -  it's the core feature for multi-caregiver households. one person acknowledges, everyone else sees that it's handled.

### 4. patients feel surveilled and want control

this came up in every patient interview without prompting. patients  -  especially younger ones  -  feel like the CGM already makes their disease visible to everyone, and the Follow app amplifies it.

> "I know my mom means well but she'll text me 'your sugar is 210' and it's like... I know. I just ate. Can I have 20 minutes before you freak out?"  -  P5, 21yo with T1D

> "It's my disease. I want to share my data but I also want to be able to say 'not right now.'"  -  P4, 45yo with T2D

**implication:** patients need a "do not disturb" or "I'm aware" button. if the patient acknowledges a reading themselves, caregivers should see "Kevin is aware of this reading" instead of getting an alert.

### 5. overnight lows are the highest-anxiety scenario

every caregiver ranked overnight lows as their #1 fear. this is clinically justified  -  nocturnal hypoglycemia is genuinely dangerous and harder to detect because the patient is asleep.

> "I set an alarm for 2am to check his Dexcom. Every night. I haven't slept through the night in three years."  -  P3

> "If your system can just tell me 'he's fine tonight'  -  that alone is worth it."  -  P7

**implication:** we need an overnight monitoring mode with faster escalation for lows. also considering a "good morning" summary: "overnight was uneventful, glucose stayed in range" to preempt the morning check.

### 6. trust takes time but breaks instantly

in concept testing, participants were cautiously optimistic about the alert system. but every single person asked some version of "what if it misses something?"

> "I would use this AND still check Follow for at least a month. If it catches everything Follow would have caught, I'd start trusting it."  -  P7

> "If it misses one bad low, I'm uninstalling it. Sorry, but that's just how it is."  -  P3

**implication:** we cannot launch with less than 95% recall on urgent-tier alerts, especially lows. precision can be a bit lower (some false alarms are acceptable early on to build trust). also need a "verification mode" where the app shows its alerts alongside raw Dexcom data so users can see it's working.

---

## persona refinements

after the research, we adjusted our personas:

- **Maya (caregiver):** added compulsive checking behavior and anxiety metrics. her core need isn't "see the data"  -  it's "stop worrying when things are fine."
- **Kevin (patient):** elevated privacy and autonomy needs. he wants to participate in the system, not just be monitored by it. added the "patient self-ack" feature based on his persona.
- **Priya (long-distance caregiver):** her primary need is coordination, not data. she has Follow already. what she doesn't have is visibility into whether someone closer is responding.

---

## journey map: caregiver alert response

**trigger:** patient's glucose enters elevated range

1. **glucose reading ingested**  -  system processes new data point (invisible to user)
2. **risk classification**  -  model classifies as elevated or urgent (invisible)
3. **alert generated**  -  caregiver receives push notification on phone
4. **alert read**  -  caregiver opens notification, reads plain-english summary
5. **context check**  -  caregiver looks at trend (is it rising? falling? been here for a while?)
6. **decision**  -  caregiver decides: ack and monitor, contact patient, or escalate
7. **acknowledge**  -  caregiver taps "I'm on it"  -  other caregivers see this
8. **follow-up**  -  caregiver checks back in 30 min to see if glucose returned to range
9. **resolution**  -  glucose returns to range, system sends "resolved" notification

**pain points in current flow (without TAN-1):**
- step 3: no alert  -  caregiver has to be actively checking
- step 5: no context in Dexcom Follow beyond the number and arrow
- step 7: no ack mechanism  -  everyone worries independently
- step 9: no resolution signal  -  caregiver keeps checking

---

## design implications

1. **alerts should include context, not just the reading.** "Blood sugar has been above 250 for 2 hours" is infinitely more useful than "Blood sugar: 267."

2. **the ack button must be zero-friction.** one tap from the push notification, no app open required. we tested 3 flows in Figma and the inline notification action won by a mile.

3. **patients need a self-acknowledge mechanism.** if the patient says "I know, I'm handling it," caregivers should see that instead of getting an alert.

4. **build trust through transparency.** show users the raw data alongside our interpretation for the first 30 days. let them verify we're catching what matters.

5. **overnight mode needs special treatment.** faster escalation, lower thresholds for lows, and a morning "all clear" message to preempt compulsive checking.

6. **less is more.** the product succeeds when caregivers check it *less*, not more. every design decision should be evaluated against: "does this reduce anxiety or increase it?"
