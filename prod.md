# AstraSave — Product Requirements (`prod.md`)

## Product
**AstraSave** is a hostel-mess meal RSVP and credits web app. It helps the mess manager estimate meal quantities before the 9 PM cutoff, reduce food waste, and avoid shortages.

## Problem
The 1,500-student mess cooks for about 85% of enrolment, while attendance is approximately 55% at breakfast, 70% at lunch and 80% at dinner. The case estimates 398 kg/day of waste, including 300 kg cooked but never served, worth roughly ₹7.2 lakh/month at ₹60/kg. Target: halve waste this semester without anyone going hungry. Figures are illustrative case assumptions.

## Goals and success metrics
- **Food waste:** 398 kg/day baseline (test week 495); target ≤199 kg/day on a 7-day average.
- **On-time RSVP:** baseline 0%; target ≥55% by week 4.
- **Forecast error:** baseline 24%; target ≤8% per meal, weekly.
- **Shortages:** target 0 per month.
- **Credits efficiency:** credits ≤40% of gross savings (modelled default 34%).
- **Net savings:** target ≥₹2 lakh/month for the full mess (modelled default ₹2.20 lakh).
- **Plate waste:** baseline 98 kg/day; target ≤75 kg/day.
- **Late passes:** proposed target ≤10 per 100 opt-outs.
- **Manager workload:** zero required data-entry fields per day; overrides optional.

## Users
- **Suresh, mess manager (primary):** needs meal counts and cooking quantities by 9 PM for the 6 AM vegetable run.
- **Hostel student:** views tomorrow’s menu, answers Eat/Skip per meal, may select a skip reason, earns credits and may redeem a reward.
- **Door staff:** selects/types student ID, checks RSVP status, records service and can apply a late pass.
- **Dean (read-only):** reviews weekly cooked/eaten/wasted kg, savings, response rate and forecast error.

## Functional requirements
### Student (R1–R8)
- R1: Show tomorrow’s menu for breakfast, lunch and dinner.
- R2: One-tap Eat or Skip for each meal.
- R3: Repeat the previous pattern for tomorrow in one tap.
- R4: Optional one-tap reason after Skip: menu, class/lab, eating out, other.
- R5: Tomorrow’s RSVPs lock at 9 PM; afterwards, changes only via a door late pass.
- R6: Show credit balance, pending credits and progress to the 15-credit reward threshold.
- R7: At 15 credits, allow redemption and deduct credits.
- R8: Show today’s locked answers and whether each credit was confirmed or voided.

### Manager (R9–R14)
- R9: Per meal show confirmed eaters, confirmed skips, non-responders and cook quantity in plates and kg.
- R10: Cooking quantity must be at least confirmed eaters × 0.4 kg and normally no more than the standard 510 kg batch; the popular-meal flag may raise the cap.
- R11: Select +5%, +8% or +10% RSVP buffer; toggle exam week and popular dish; update all numbers immediately.
- R12: Optional quantity override within ±15%, with a reason; the minimum confirmed-eater floor still applies.
- R13: Show a simulated WhatsApp-style kitchen summary (no actual sending).
- R14: Manager sees aggregate counts only, never student names or individual reasons.

### Door check and credits (R15–R21)
- R15: Selecting/typing an ID returns green (Eat), amber (Skip) or grey (no RSVP) and records service.
- R16: A student who skipped may be served using a late pass; void the opt-out bonus and reduce reliability by 1.
- R17: One service per student per meal; repeat scan says “Already served”.
- R18: Never refuse food. Late-pass and no-RSVP walk-ins are served after RSVP students and logged.
- R19: One credit for each on-time RSVP; opt-out bonus: breakfast +3, lunch +1, dinner +0.
- R20: Log credit settles at 9 PM; opt-out bonus settles after meal close only if the student was not served.
- R21: Reliability starts at 5, drops by 1 per late pass, blocks bonus credits below 3, and recovers by 1 per 5 honoured opt-outs (maximum 5).
- IMPORTANT: Only consider which option chosen at meal Lock, and add only that particular amount to pending reward. eg: if Skip was chosen first, then changed to Eat (before lock), only add the rewards from Eat into the pending amount (don't add Skip's reward)

### Impact and demo (R22–R28)
- R22: Show cooked/eaten kg, waste today vs product, kg and ₹ saved, credits paid, net savings and shortages.
- R23: Impact controls for response rate, buffer, menu effect and credit values; all results update.
- R24: Weekly Dean report with cooked/eaten/wasted kg, savings, response rate and forecast error.
- R25: Demo clock can jump to 8:45 PM, 9:01 PM and the next morning.
- R26: Persist state in browser; Reset demo restores seed.
- R27: Every specified edge case must behave as documented in the PRD/TRD.
- R28: Student and manager views work at 390 px width; desktop also works.

## Credit rules
- 1 credit = ₹1; each on-time Eat or Skip RSVP earns 1 log credit.
- Skip bonus: +3 breakfast, +1 lunch, +0 dinner. Total for a skipped meal: 4/2/1 credits respectively; an eaten meal earns 1.
- Maximum 7 credits per student per day.
- 15 credits redeem one pilot reward (proposed ice-cream cone); deduct 15 and show a one-time voucher code.
- Only the answer at lock counts; toggling before lock earns no extra credits.
- Late pass voids the bonus for that meal, keeps the log credit and reduces reliability by 1.
- Below reliability 3: no bonus, but log credit remains. Every 5 honoured opt-outs restores 1 reliability up to 5.
- Any skipped-meal credit total stays below one plate’s cost of ₹24 (0.4 kg × ₹60).
- At 55% response, the PRD estimates 3,837 credits/day, about ₹1.15 lakh/month, around 34% of ₹3.35 lakh gross savings, and approximately ₹2.20 lakh net savings. These are pilot assumptions.

## Edge cases (must preserve)
- E1 skip then turn up: amber, late pass, bonus voided, log credit retained, reliability −1.
- E2 Eat RSVP but no-show: no penalty/extra credit; log for weekly report.
- E3 after-lock answer changes blocked; only late pass at door.
- E4 no response: non-responder included in forecast; no credit.
- E5 last pre-lock tap wins; credits use the answer at lock.
- E6 exam week: expected non-responder attendance multiplied by 0.9.
- E7 popular meal: recommendation and cap multiplied by 1.10 for that meal.
- E8 override within ±15%, with reason; never below confirmed-eater floor; log it.
- E9 no history: use seed attendance 55%/70%/80%.
- E10 zero RSVPs: forecast from non-responders with 15% buffer.
- E11 all responders skip: cook for non-responders; show “unusually low”.
- E12 confirmed eaters exceed batch: floor wins over cap; show “above standard batch”.
- E13 reliability below 3: no bonus, log credit paid, explain recovery.
- E14 ≥15 credits: redeem, deduct 15 and show one-time code.
- E15 <15 credits: disable redemption and show amount still needed.
- E16 no-RSVP walk-in: grey, serve from safety portion after RSVP students, no credit.
- E17 duplicate scan: “Already served”, no second record.
- E18 unknown ID: “ID not found”; do not refuse food; serve/log walk-in.
- E19 outside serving window: “Meal not open”, no record.
- E20 missing menu: show “Menu not published yet”; RSVP remains possible.
- E21 meal closes: settle bonuses; block further scans.
- E22 mess closed (Could): no RSVPs, credits or cooking quantity.
- E23 refresh restores browser state; reset restores seed.
- E24 manager/Dean views show counts only; names only at door for scanned student.
- E25 opting out never costs money or access; only late pass affects reliability.
- E26 use demo clock only, never device time.
- E27 changing response-rate control regenerates anonymous counts and updates all results.
- E28–E30: retain any additional cases defined in the companion TRD/spec; do not invent behaviour where not specified.

## Non-goals
No real accounts/login, backend/database/API, ML forecast, real payments/vouchers, SMS/WhatsApp sending, camera/QR/hardware scanning, mobile-native app, procurement/inventory integration, NGO pickups/2-hour surplus workflow, menu voting/leaderboard, per-student manager analytics, fines/charges, changing the ₹4,200 monthly fee, or learning no-show rates into the forecast. The NGO button is disabled “next step”; staff summary and voucher are simulated.

## Pilot phases (proposals to confirm)
- **P0, demo:** core click path works and check numbers match.
- **P1, 2 weeks:** breakfast-first pilot; compare one-credit vs zero-credit groups. Go if response ≥40% by week 2, breakfast waste falls ≥30% vs prior two weeks, and shortages are zero.
- **P2, 2 weeks:** all meals in same scope. Go if net savings positive, forecast error ≤8%, and credits ≤40% of gross savings.
- **P3, 4 weeks:** all five blocks. Continue if 7-day waste average ≤199 kg/day and shortages remain zero.

## Risks and open questions
Risks include low response, credit gaming/cost, shortages, manager adoption, inaccurate odd-day forecasts, menu-driven skips and overclaimed savings. Use non-responder buffer, scan/late-pass/reliability rules, floor/buffer/override/shortage log, minimal manager input, menu preview/reasons, visible assumptions and sensitivity controls. Open items include the 9 PM cutoff, pilot credit/reward value, funding split, per-block weighing feasibility, final survey sample/results, student ID source and acceptance of reliability rules. Values are pilot proposals, not confirmed facts.
