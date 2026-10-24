# AstraSave — Technical Requirements (`tech.md`)

## Non-negotiable constraints
- Use the station’s existing web template (e.g. Next.js or Vite) as provided, with TypeScript.
- One browser-only React app; **no backend, server code, database, API, authentication, external integrations, or new libraries**.
- One React context mirrored to `localStorage` under key `astrasave.v1`.
- Nothing leaves the browser. Share summary is a text modal; reward redemption displays a code.
- Use a stateful demo clock only. Never read the device clock.
- Mobile-first at 390 px; desktop must work.
- Units: ₹, kg (one decimal), plates (whole numbers); 1 plate = 0.4 kg.
- Build exactly the listed files. No test framework, README, refactors or extra features. Use plain CSS bars or inline SVG for charts.

## Architecture
Four views (Student, Manager, Door check, Impact) → React context/store → pure calculation engine. Seed data initializes first load and Reset demo. Views dispatch actions and render returned values; **all displayed numbers come from `engine.ts`; components do no arithmetic.**

## Required files
- `src/lib/types.ts` — shared types.
- `src/data/seed.ts` — all settings, seed data and anonymous RSVP-count generator.
- `src/lib/engine.ts` — pure functions only: forecast, cooking quantity, waste, credits, reliability and door-check result. No React, clock reads or storage.
- `src/lib/clock.ts` — demo clock, lock test, meal windows, five presets and label formatting.
- `src/lib/selfcheck.ts` — run numeric scenarios against engine and return pass/fail per row.
- `src/lib/store.tsx` — React context, localStorage persistence and all actions: RSVP, scan, late pass, redeem, override, settings, clock and reset.
- `src/components/AppShell.tsx` — role tabs, demo clock, Reset demo, Run self-check.
- `src/components/StudentView.tsx` — R1–R8.
- `src/components/ManagerView.tsx` — R9–R14.
- `src/components/DoorCheck.tsx` — R15–R18.
- `src/components/ImpactView.tsx` — R22–R24.

## Types
```ts
type MealId = 'breakfast' | 'lunch' | 'dinner';
type Choice = 'eat' | 'skip';
type Reason = 'menu' | 'class' | 'eating-out' | 'other';
type Rating = 'liked' | 'okay' | 'disliked';

interface Settings {
  enrolment: number; plateKg: number; costPerKg: number; daysPerMonth: number;
  batchCapKg: number; responseRate: number; bufferRsvp: number;
  bufferNonResp: number; menuCut: number; examWeek: boolean;
  popularMeal: MealId | null; monthlyFeeRs: number; payoutShare: number;
  feedbackTakeUp: number; feedbackRewardRs: number; feedbackUnlock: number;
  redeemAtRs: number; overridePct: number; lockHour: number;
}
interface MealSeed {
  attendance: number; plateWasteKg: number; opensAt: string; closesAt: string;
  history: number[];
}
interface Student {
  id: string; name: string; block: string; balanceRs: number; pendingRs: number;
  reliability: number; honouredOptOuts: number; feedbackThisMonth: number;
}
interface Rsvp { studentId: string; day: number; meal: MealId; choice: Choice; reason?: Reason }
interface Scan {
  studentId: string; day: number; meal: MealId;
  outcome: 'green' | 'amber' | 'grey'; latePass: boolean;
}
interface RewardEvent {
  studentId: string; day: number; meal: MealId;
  kind: 'pending' | 'paid' | 'voided' | 'withheld' | 'redeem'; amountRs: number;
}
interface Feedback { studentId: string; day: number; meal: MealId; rating: Rating }
interface Override { day: number; meal: MealId; suggestedKg: number; usedKg: number; reason: string }
interface Shortage { day: number; meal: MealId; note: string }
interface Clock { day: number; minute: number }
interface AppState {
  settings: Settings; students: Student[]; rsvps: Rsvp[]; scans: Scan[];
  rewardEvents: RewardEvent[]; feedback: Feedback[]; overrides: Override[];
  shortages: Shortage[]; walkIns: {day:number; meal:MealId; id:string}[];
  anonymous: Record<number, Record<MealId, {eat:number; skip:number}>>;
  closedMeals: string[]; clock: Clock;
  role: 'student' | 'manager' | 'door' | 'impact'; currentStudentId: string;
}
```

## Seed and defaults
- Defaults: enrolment 1500; plate 0.4 kg; ₹60/kg; 30 days/month; batch cap 510 kg/meal; response rate 0.55; RSVP buffer 0.08; non-response buffer 0.15; menu effect 0; exam week false; popular meal null; monthly fee ₹4,200; payout share 0.20; feedback take-up 0.30; month-end reward ₹20; unlock at 5 feedback entries; redeem threshold ₹50; override limit 0.15; lock hour 21.
- Start state: day 0, 6:00 PM, Student role, current student S001.
- Meals: breakfast 55%, 26 kg plate waste, 07:00–09:30; lunch 70%, 34 kg, 12:00–14:30; dinner 80%, 38 kg, 19:30–21:30.
- Even days use menu A; odd days menu B. Day 0 is A, tomorrow B. A: breakfast idli/sambar/chutney; lunch rice/rajma/jeera aloo/salad; dinner chapati/paneer butter masala/dal/rice. B: breakfast poha/boiled egg/tea; lunch rice/sambar/cabbage poriyal/curd; dinner veg biryani/raita/gulab jamun.
- Fourteen-day attendance histories: breakfast alternates 57%/53%, lunch 72%/68%, dinner 82%/78%, seven each; means equal seed attendance.
- Anonymous counts use round-half-up: `eat = round(1500 × responseRate × attendance)` and `skip = round(1500 × responseRate × (1 − attendance))`. Subtract that day’s demo-student RSVPs per meal (never below zero), so totals including demo students match formula. Generate day 0/day 1 at seed and later days when needed. At 55%: breakfast 454 eat/371 skip; lunch 578/248; dinner 660/165. Non-responders = enrolment − eat − skip.
- Exactly eight demo-student records (S001–S008), per the PRD/TRD seed table. Only day-0 dinner has seeded RSVPs; none tomorrow. Preserve the individual seed values and scenario paths in the source TRD.

## Core engine formulas
For meal `m`, attendance `a`, eat/skip counts and `n = 1500 − eat − skip`:
- `expNonResp = n × a × fExam`, where `fExam = 0.9` in exam week, otherwise `1`.
- `raw = 0.4 × (eat × (1 + bufferRsvp) + expNonResp × (1 + bufferNonResp)) × fPop`.
- `cook = max(0.4 × eat, min(510 × fPop, raw))`, where `fPop = 1.10` for the flagged popular meal, otherwise `1`. Confirmed-eater floor wins over cap.
- Plates = ceiling of cook kg ÷ 0.4. An override must be within `max(floor, 0.85 × cook)` and `1.15 × cook`; when set, use override in downstream calculations.
- Simulated actual eaters = `1500 × a`, unchanged by controls.
- `unserved = max(0, cook − 0.4 × actual)`; `shortage = max(0, 0.4 × actual − cook)`.
- `waste_m = unserved + plateWasteKg_m × (1 − menuCut)`.
- `wasteToday_m = max(0, 510 − 0.4 × actual) + plateWasteKg_m`.
- `grossMonthly = (sum(wasteToday) − sum(waste)) × 60 × 30`.
- Usual attendance is mean of 14-day history; skip likelihood `s = 1 − a`; reward rate `p = payoutShare × s`; meal reward `r = round(p × mealValue, 2)`, where meal value = `4200 / 30 / 3 = ₹46.67`.
- Daily opt-out rewards = sum of `skip_m × r_m` for opt-outs who stay away. Month-end rewards = `enrolment × feedbackTakeUp × feedbackRewardRs`. Net monthly = gross − daily rewards × 30 − month-end rewards.
- Show reduction % = saved ÷ total wasteToday, and rewards as a share of gross.

## Clock, credits and door rules
- Tomorrow accepts RSVPs before current day’s 21:00 lock; current-day meals are read-only. Manager defaults to tomorrow at/after 21:00, otherwise today; before lock, tomorrow shows “Not locked yet, live preview”.
- At lock, each demo student’s Skip creates a pending reward and increases pending amount. At meal close: skipped and not served + reliability ≥3 → pay reward to balance, increment honoured opt-outs; at 5, reliability +1 (max 5) and reset count. Late-pass service voids reward. Reliability below 3 withholds bonus. Pending and balance remain separate.
- Late pass: reliability −1 (minimum 0), honoured-opt-out count reset, scan marked late pass, bonus voided at close.
- Redeem at ₹50 or more: deduct ₹50, record redemption and show code formed from student number and day (e.g. `S004-D0`).
- Feedback: one rating per student/meal/day on a closed meal; ignore duplicates. Fifth monthly entry unlocks the month-end reward, without moving money.
- Door outcomes: green for Eat, amber for Skip, grey for no RSVP; duplicate gives “Already served”; unknown ID gives “ID not found”; outside serving window gives “No meal open”.

## Self-check and acceptance
- Built-in Run self-check; compare all TRD section 7 scenarios with tolerance 0.1 kg and ₹1. At default, reference rows include total cook 1,343.6 kg, waste 211.6 kg, reduction 46.8%, gross ₹335,471, rewards ₹85,835, net ₹249,637 (small displayed rounding differences follow the TRD).
- Check numbers and edge cases are defined in the source TRD; use it as authoritative for every row. Never substitute guessed values.
- Preserve privacy: Manager and Impact/Dean views never show student names or individual reasons.
- Reset returns every screen to the same seed state.

## Build sequence and token discipline
Use four prompts/milestones:
1. Types, seed, engine, clock, self-check; establish check-number baseline.
2. Store, app shell and Student view; validate student click path and lock.
3. Manager and Impact views; validate toggles, override, share modal and impact values.
4. Door check, credit settlement, redemption and required edge cases; self-check again.
Each prompt has one gate. Do not rebuild files that pass. Ask for targeted diffs naming only the file/function and failing rule ID. At most two corrections; if time runs short cut in this order: weekly report/copy text, reason chips, shortage logging/share-modal polish, impact chart (keep tiles), repeat button. Never cut engine, 9 PM lock, credits, door check or self-check.

## Do not build
No login/signup/password roles, backend/database/API/network calls, real QR/ID/camera scan, real WhatsApp/SMS/push, payments or real vouchers, multiple hostels/admin panels, manager per-student analytics, ML, animations, dark mode or settings pages.
