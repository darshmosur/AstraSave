# AstraSave — UI/UX Requirements (`ui.md`)

## Design objective
A clean, minimal interface that lets a student answer in about three seconds and lets the manager read the key number immediately.

**Keep the UI strictly limited to the screens, labels, controls and microcopy specified in this document and `prod.md`/`tech.md`. Do not add extra or unnecessary text, explanatory paragraphs, decorative copy, new sections, invented tooltips, extra labels or features. Do not invent wording. Use the exact microcopy below. If a text element is not specified, omit it.**

## Principles
1. One screen, one job: Student answers; Manager reads cooking quantities; Door returns a service status; Impact demonstrates savings.
2. One tap per decision. Eat/Skip requires no second screen, form or sign-in.
3. Every status communicates with both colour and words.
4. Large, plain, accessible controls: buttons at least 44 px tall; body text at least 16 px.
5. Numbers are supplied by the engine; views do not calculate.

## Colour tokens
Define once in `src/theme.css` and use only these variables everywhere. No other hard-coded hex values.
```css
:root {
  --primary: #6C5CE7;
  --primary-dark: #4833A8;
  --primary-mid: #8073EA;
  --primary-soft: #A79DF0;
  --primary-light: #C4BDF5;
  --tint: #E1DEF9;
  --tint-soft: #EEEBFB;
  --surface: #F8F6FD;
  --bg: #FFFFFF;
  --ink: #000000;
  --muted: #686868;
  --grey: #ACACAC;
  --good: #1E7F54;
  --warn: #F2B84B;
  --bad: #C0392B;
}
```
- Primary: `--primary`; dark cards/net saving: `--primary-dark`; secondary chart bars/hover: `--primary-mid`; chart bars/selected borders: `--primary-soft`; light chart bars/progress track: `--primary-light`; chips/borders: `--tint`; quiet fills/stripes: `--tint-soft`; panels: `--surface`; background white; text black; secondary text `--muted`; waste-today bar/disabled/no RSVP `--grey`.
- Status colours only for door result, lock banner and shortages: good `--good` with white text; warning `--warn` with black text; error/locked `--bad` with white text. Grey statuses use black text.
- Never use white text on `--primary-mid`. Maintain at least 4.5:1 contrast.

## Typography and layout
- Font: `system-ui, Arial, sans-serif`; do not load fonts or images from network.
- Body 16 px; captions 14 px; card titles 20 px; screen titles 24 px; key numbers 32 px and bold.
- Body weight 400; titles 600; buttons and numbers 700.
- 4 px spacing grid; 16 px card padding and page margin; 12 px between cards.
- Cards radius 12 px; buttons/chips radius 8 px.
- Touch targets ≥44 × 44 px, with 8 px between neighbours.
- Mobile first at 390 px: one column, top bar then content, no horizontal scrolling.
- Desktop content max width 1100 px, centred. Manager and Impact become two columns above 900 px. Student and Door stay one column, max width 480 px, centred.
- Top bar: primary background, white text; role tabs, demo clock, Reset demo and Run self-check. Active role is a white pill with dark-primary text.
- Motion: none except 150 ms fade on lock banner. No other animation.

## Reusable components
- **Eat/Skip choice:** side-by-side, 48 px tall. Chosen Eat is primary fill; chosen Skip is dark-primary fill; white text. Unchosen is primary outline. Disabled is grey with no fill. Visible 2 px ink focus ring.
- **Primary button:** primary fill, white text, 48 px tall; pressed dark-primary.
- **Secondary button:** primary outline and text, no fill.
- **Meal card:** white, 1 px tint border; title, menu line, serving window and actions. Current meal gets a 3 px primary top border and “NOW”; closed meal has grey title and “Closed” chip.
- **Status chip:** pill, 14 px text; default tint fill/dark-primary text. Door status uses status colours.
- **Banner:** full-width strip within card, 16 px text, no icon. Info tint; caution warning; locked red with white text.
- **Tile:** surface fill, large number, muted label. Net-saving tile dark-primary with white text.
- **Progress bar:** 12 px; primary-light track and primary fill; label above.
- **Reason chips:** four wrapping pills: Menu, Class or lab, Eating out, Other. Unselected tint; selected primary with white text.
- **Modal:** centred white card on dark overlay, close button and copyable text.
- **Table:** tint-soft stripes, 16 px text; primary header with white text.
- **Bar chart:** grouped CSS bars, value above each bar.

## Screen layouts (top to bottom at 390 px)

### Shared top bar
Role tabs: Student, Manager, Door, Impact. Demo clock and buttons: Reset demo, Run self-check.

### Student
1. Student strip: name, block, “Reliability 5 of 5”, selector for eight demo students.
2. Day tabs: Tomorrow (default), Today.
3. Lock banner with countdown or locked state.
4. Three meal cards: menu, serving window, Eat/Skip, reward line under Skip.
5. Optional reason chips shown only after Skip.
6. Full-width secondary “Repeat today’s answers” button.
7. Rewards panel: balance ₹, pending ₹, progress to ₹50, Redeem, feedback count/progress.
8. Today tab: read-only locked answers/status and one-tap feedback row for closed meals.

### Manager
1. Day switch: Today/Tomorrow; Exam week and Popular dish toggles.
2. Three meal cards with usual attendance (last 14 days), eat/skip/non-response counts, plates/kg and buffer selector. Serving meal is enlarged and marked NOW; plates/kg are the largest figures (32 px bold, dark-primary).
3. Banners inside cards: live preview, above standard batch, unusually low.
4. Optional adjustment inside card: kg field, reason and allowed range.
5. One dark-primary “Estimated food saved” tile for selected day.
6. Actions: “Share with kitchen”, “Log shortage”, disabled “Contact NGO (next step)”.
7. Desktop: meal cards side-by-side, tile/actions in right column.

### Door check
1. Meal strip with open meal and serving window, or “No meal open”.
2. Demo student selector, free-text ID input, primary Scan button.
3. Full-width result card in status colour, 24 px title and action.
4. Four small tiles: Served, plates left, late passes, walk-ins.
5. Striped table of last ten scans, ID and outcome.

### Impact / Dean
1. Tiles in two columns at 390 px: waste today vs product, % saved, gross ₹, opt-out credits ₹, month-end rewards ₹, net ₹, rewards as share of gross, shortage kg. Net saving is largest and dark-primary.
2. Grouped CSS bars by meal: today vs product, grey vs primary.
3. KPI table rows from Metrics Plan, value, target and pass/amber status chips.
4. Controls with adjacent values: response rate, buffer, menu effect, payout share, feedback take-up, month-end reward; “Reset to defaults”.
5. Dean report: weekly/monthly toggle, one row per day/week, “Copy as text”.
6. Never display student names or individual reasons on Manager or Impact/Dean views.

## Exact microcopy
Use the following strings verbatim; interpolate only the values in braces.

| Where | Exact text |
|---|---|
| Lock before | `Locks at 9:00 PM tonight, in {time left}` |
| Lock after | `Locked at 9:00 PM. At the door, ask for a late pass` |
| Eat caption | `Eat: you are counted in. 1 log credit awarded` |
| Skip caption | `Skip before 9 PM and stay away: earn ₹{reward} after the meal` |
| Reward pending | `₹{amount} pending until the meal closes` |
| Reward paid | `₹{amount} paid` |
| Reward voided | `Voided: you were served with a late pass` |
| Reliability low | `Reliability {n} of 5. Honour 5 opt-outs to earn rewards again` |
| Redeem short | `₹{amount} more to redeem` |
| Redeem ready | `Redeem ₹50 voucher` |
| Voucher | `Your code: {code}. Show it at the counter` |
| Feedback prompt | `How was {meal}? Liked it, It was okay, Did not like it` |
| Feedback progress | `{n} of 5 feedback entries for the month-end reward` |
| Feedback unlocked | `Month-end reward unlocked: a small treat from the mess` |
| Menu missing | `Menu not published yet. You can still answer` |
| Manager live preview | `Not locked yet, live preview` |
| Manager above batch | `Above standard batch: confirmed eaters need more than usual` |
| Manager low | `Unusually low: almost everyone has opted out` |
| Usual attendance | `Usually {pct}% eat {meal} (last 14 days)` |
| Override error | `Allowed range {low} to {high} kg` |
| Door green | `Opted in, serve` |
| Door amber | `Opted out, offer late pass` |
| Door grey | `No RSVP, serve from the safety portion` |
| Door repeat | `Already served` |
| Door unknown | `ID not found. Serve and log as walk-in` |
| Door closed | `No meal open` |
| NGO button | `Contact NGO (next step)` |
| Demo clock | `{Weekday} {date}, {time}` |

## Accessibility and acceptance checklist
- Every button/tab ≥44 px tall and 8 px neighbour gap.
- Body text ≥16 px; important text never below 14 px.
- Status has word + colour; contrast ≥4.5:1; no white text on `--primary-mid`.
- Eat/Skip keyboard-accessible with visible focus ring.
- Student and Door fit 390 px without horizontal scroll.
- Only `src/theme.css` colour variables; no other hex values.
- Every microcopy string above appears exactly.
- Manager/Impact never show names or individual reasons.
- No animation except 150 ms lock fade; no network fonts/images.
- Reset returns every screen to identical seed state.
