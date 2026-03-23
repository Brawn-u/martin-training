# Exercise Expansion & Swap Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand the exercise library in Martin Training from ~5 to ~12 exercises per category/tier, and improve the swap button to hide when no alternatives exist.

**Architecture:** All changes are in a single file — `index.html`. The file uses CDN React with Babel standalone (no build step). Exercise data lives in three global objects (`EX`, `DEMO`, `VIDEO_MAP`) defined in a `<script type="text/babel">` block starting around line 27. The workout is rendered in the `App` component via a `useMemo`. The swap button calls `shuffleSingle(idx)` which increments `exSeeds[curDay-idx]`; the useMemo re-runs and picks a new exercise using that seed.

**Tech Stack:** React 18 (CDN UMD), Babel standalone, plain HTML/CSS — no npm, no build step, open in a browser to test.

**Spec:** `docs/superpowers/specs/2026-03-22-exercise-expansion-design.md`

---

## Codebase Map

All changes are in one file: `index.html`

| Section | Approx lines | What it is |
|---|---|---|
| `VIDEO_MAP` | ~36 | Object mapping exercise name → YouTube video ID |
| `DEMO` | ~38 | Object mapping exercise name → `{ desc, cues[] }` |
| `EX.strength` | ~40 | Strength exercise pools by category + equipment tier |
| `EX.hiit` | ~42 | HIIT exercise pools (crossfit aliases hiit; bands aliases strength.minimal) |
| `App` component | ~112–end | React component; `workout` useMemo at ~132, swap button at ~223 |

**Key data shape:**
```js
// Exercise pool entry
{ name: "Exercise Name", muscle: "Muscle/Groups", avoid: ["BodyArea1"] }

// DEMO entry
"Exercise Name": { desc: "One sentence description", cues: ["Cue 1", "Cue 2", "Cue 3", "Cue 4"] }

// VIDEO_MAP entry
"Exercise Name": "youtubeVideoId"

// Valid avoid values (must match BODY_AREAS exactly):
// "Shoulders", "Upper Back", "Lower Back", "Knees", "Hips", "Wrists", "Ankles", "Neck", "Elbows"
```

**Key functions:**
- `filt(list, avoid)` — filters exercises whose `avoid[]` intersects the user's active avoid list
- `pick(list, n, rng)` — shuffles with seeded rng, returns first n items
- `seededRandom(seed)` — returns an rng function from integer seed
- `getCatForSlot(dayType, idx)` — returns category name for a workout slot
- `shuffleSingle(idx)` — adds a random offset to `exSeeds[curDay-idx]`, triggering useMemo re-pick

**`EX.crossfit = EX.hiit` and `EX.bands` is aliased from `EX.strength.minimal` — you only need to expand `EX.strength` and `EX.hiit` directly.**

---

## Note: Shuffle Diversity & Swap Cycling Are Already Implemented

Before starting, be aware of two things the existing code already handles:

**Shuffle diversity:** The "Shuffle All" button already calls `setSeed(s=>s+1)` — each tap produces a new seed and therefore a different workout. The diversity problem is purely about pool size, which Tasks 1–6 fix. No new state or logic needed.

**Swap cycling:** `shuffleSingle(idx)` already uses `(p[key]||0) + Math.floor(Math.random()*10000) + 1` — each tap changes the seed for that slot, producing a different exercise. Task 9 only needs to add the `swappable` flag (hide button when pool is empty) and cross-slot exclusion. The cycling mechanism is already correct. **Spec Sections 2 (Selection & Cycling / swapCounts redesign) and 3 (Shuffle Diversity Fix / shuffleCount state) are intentionally superseded by the existing implementation — do not introduce new state for these.**

---

## Task 1: Expand EX.strength.upper_push

**File:** `index.html` — find `EX.strength.upper_push` object

- [ ] **Step 1: Locate the upper_push section**

Search for `upper_push:{full:[` in `index.html`. This is inside the `EX` object assigned around line 40.

- [ ] **Step 2: Add exercises to upper_push.full**

Append these entries to the `full` array inside `EX.strength.upper_push`:
```js
{name:"Close Grip Bench Press",muscle:"Chest/Triceps",avoid:["Elbows","Wrists"]},
{name:"Weighted Dip",muscle:"Chest/Triceps/Shoulders",avoid:["Shoulders","Elbows","Wrists"]},
{name:"Cable Lateral Raise",muscle:"Shoulders",avoid:["Shoulders"]},
{name:"Pec Deck",muscle:"Chest",avoid:["Shoulders","Elbows"]},
{name:"Tricep Pushdown",muscle:"Triceps",avoid:["Elbows","Wrists"]},
{name:"Dumbbell Chest Fly",muscle:"Chest",avoid:["Shoulders","Elbows"]}
```

- [ ] **Step 3: Add exercises to upper_push.home**

Append to the `home` array inside `EX.strength.upper_push`:
```js
{name:"Dumbbell Lateral Raise",muscle:"Shoulders",avoid:["Shoulders"]},
{name:"Diamond Push-Up",muscle:"Triceps/Chest",avoid:["Elbows","Wrists","Shoulders"]},
{name:"Dumbbell Chest Fly",muscle:"Chest",avoid:["Shoulders","Elbows"]},
{name:"Pike Push-Up",muscle:"Shoulders",avoid:["Shoulders","Wrists"]}
```

- [ ] **Step 4: Add exercises to upper_push.minimal**

Append to the `minimal` array inside `EX.strength.upper_push`:
```js
{name:"Dumbbell Lateral Raise",muscle:"Shoulders",avoid:["Shoulders"]},
{name:"Band Lateral Raise",muscle:"Shoulders",avoid:["Shoulders"]},
{name:"Diamond Push-Up",muscle:"Triceps/Chest",avoid:["Elbows","Wrists","Shoulders"]},
{name:"Band Overhead Press",muscle:"Shoulders/Triceps",avoid:["Shoulders","Wrists"]}
```

- [ ] **Step 5: Open index.html in a browser, generate a workout, verify**

Go to Setup → Full Gym → Strength → Generate. Look at the workout — push exercises should now occasionally show the new names. Reshuffle a few times to confirm variety.

- [ ] **Step 6: Commit**
```bash
git add index.html
git commit -m "feat: expand upper_push exercise pool"
```

---

## Task 2: Expand EX.strength.upper_pull

**File:** `index.html` — find `upper_pull:{full:[` inside the `EX` object

- [ ] **Step 1: Add exercises to upper_pull.full**

Append to the `full` array:
```js
{name:"T-Bar Row",muscle:"Back/Biceps",avoid:["Lower Back","Wrists"]},
{name:"Chest-Supported Row",muscle:"Back/Biceps",avoid:["Wrists"]},
{name:"Straight Arm Pulldown",muscle:"Lats",avoid:["Shoulders"]},
{name:"Hammer Curl",muscle:"Biceps/Forearms",avoid:["Elbows"]},
{name:"Preacher Curl",muscle:"Biceps",avoid:["Elbows","Wrists"]},
{name:"Cable Bicep Curl",muscle:"Biceps",avoid:["Elbows","Wrists"]}
```

- [ ] **Step 2: Add exercises to upper_pull.home**

Append to the `home` array:
```js
{name:"Dumbbell Hammer Curl",muscle:"Biceps/Forearms",avoid:["Elbows"]},
{name:"Dumbbell Bicep Curl",muscle:"Biceps",avoid:["Elbows","Wrists"]},
{name:"Chest-Supported Dumbbell Row",muscle:"Back/Biceps",avoid:["Wrists"]},
{name:"Dumbbell Rear Delt Fly",muscle:"Rear Delts",avoid:["Shoulders"]}
```

- [ ] **Step 3: Add exercises to upper_pull.minimal**

Append to the `minimal` array:
```js
{name:"Dumbbell Hammer Curl",muscle:"Biceps/Forearms",avoid:["Elbows"]},
{name:"Band Bicep Curl",muscle:"Biceps",avoid:["Elbows"]},
{name:"Band Face Pull",muscle:"Rear Delts/Upper Back",avoid:["Shoulders"]}
```

- [ ] **Step 4: Verify in browser**

Generate a workout. Pull exercises (back/biceps) should show new variety.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: expand upper_pull exercise pool"
```

---

## Task 3: Expand EX.strength.lower_push

**File:** `index.html` — find `lower_push:{full:[` inside the `EX` object

- [ ] **Step 1: Add exercises to lower_push.full**

Append to the `full` array:
```js
{name:"Leg Extension",muscle:"Quads",avoid:["Knees"]},
{name:"Front Squat",muscle:"Quads/Glutes",avoid:["Knees","Lower Back","Wrists"]},
{name:"Smith Machine Squat",muscle:"Quads/Glutes",avoid:["Knees","Lower Back"]},
{name:"Sissy Squat",muscle:"Quads",avoid:["Knees"]},
{name:"Box Step-Up",muscle:"Quads/Glutes",avoid:["Knees","Ankles"]}
```

- [ ] **Step 2: Add exercises to lower_push.home**

Append to the `home` array:
```js
{name:"Dumbbell Sumo Squat",muscle:"Quads/Glutes/Adductors",avoid:["Knees","Wrists"]},
{name:"Reverse Lunge",muscle:"Quads/Glutes",avoid:["Knees","Ankles"]},
{name:"Wall Sit",muscle:"Quads",avoid:["Knees"]},
{name:"Single Leg Squat",muscle:"Quads/Glutes",avoid:["Knees","Ankles","Hips"]}
```

- [ ] **Step 3: Add exercises to lower_push.minimal**

Append to the `minimal` array:
```js
{name:"Dumbbell Sumo Squat",muscle:"Quads/Glutes/Adductors",avoid:["Knees","Wrists"]},
{name:"Band Lateral Walk",muscle:"Glutes/Hips",avoid:["Knees","Hips"]},
{name:"Reverse Lunge",muscle:"Quads/Glutes",avoid:["Knees","Ankles"]},
{name:"Wall Sit",muscle:"Quads",avoid:["Knees"]}
```

- [ ] **Step 4: Verify in browser**

Generate with Full Gym, check leg day for new quad exercises.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: expand lower_push exercise pool"
```

---

## Task 4: Expand EX.strength.lower_pull

**File:** `index.html` — find `lower_pull:{full:[` inside the `EX` object

- [ ] **Step 1: Add exercises to lower_pull.full**

Append to the `full` array:
```js
{name:"Good Morning",muscle:"Hamstrings/Lower Back",avoid:["Lower Back"]},
{name:"Nordic Curl",muscle:"Hamstrings",avoid:["Knees"]},
{name:"Seated Leg Curl",muscle:"Hamstrings",avoid:["Knees"]},
{name:"Glute Ham Raise",muscle:"Hamstrings/Glutes",avoid:["Knees","Lower Back"]},
{name:"Cable Kickback",muscle:"Glutes",avoid:[]}
```

- [ ] **Step 2: Add exercises to lower_pull.home**

Append to the `home` array:
```js
{name:"Donkey Kick",muscle:"Glutes",avoid:[]},
{name:"Fire Hydrant",muscle:"Glutes/Hips",avoid:["Hips"]},
{name:"Dumbbell Good Morning",muscle:"Hamstrings/Lower Back",avoid:["Lower Back"]},
{name:"Curtsy Lunge",muscle:"Glutes/Adductors",avoid:["Knees","Ankles"]}
```

- [ ] **Step 3: Add exercises to lower_pull.minimal**

Append to the `minimal` array:
```js
{name:"Donkey Kick",muscle:"Glutes",avoid:[]},
{name:"Band Kickback",muscle:"Glutes",avoid:[]},
{name:"Band Hip Thrust",muscle:"Glutes",avoid:["Lower Back"]},
{name:"Dumbbell Good Morning",muscle:"Hamstrings/Lower Back",avoid:["Lower Back"]}
```

- [ ] **Step 4: Verify in browser**

Generate a Lower Body or Full Body day, confirm new hamstring/glute exercises appear.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: expand lower_pull exercise pool"
```

---

## Task 5: Expand EX.strength.core

**File:** `index.html` — find `core:{full:[` inside `EX.strength`

- [ ] **Step 1: Add exercises to core.full**

Append to the `full` array:
```js
{name:"Cable Crunch",muscle:"Abs",avoid:["Lower Back","Neck"]},
{name:"Decline Sit-Up",muscle:"Abs",avoid:["Lower Back","Neck"]},
{name:"Ab Wheel Rollout",muscle:"Abs/Core",avoid:["Lower Back","Wrists","Shoulders"]},
{name:"Landmine Rotation",muscle:"Obliques/Core",avoid:["Lower Back","Wrists"]},
{name:"Captain's Chair Leg Raise",muscle:"Lower Abs",avoid:["Shoulders","Lower Back"]}
```

- [ ] **Step 2: Add exercises to core.home**

Append to the `home` array:
```js
{name:"Hollow Body Hold",muscle:"Core",avoid:[]},
{name:"Russian Twist",muscle:"Obliques",avoid:["Lower Back"]},
{name:"Bicycle Crunch",muscle:"Abs/Obliques",avoid:["Lower Back","Neck"]},
{name:"Superman Hold",muscle:"Lower Back/Glutes",avoid:["Lower Back"]}
```

- [ ] **Step 3: Add exercises to core.minimal**

Append to the `minimal` array:
```js
{name:"Hollow Body Hold",muscle:"Core",avoid:[]},
{name:"Bicycle Crunch",muscle:"Abs/Obliques",avoid:["Lower Back","Neck"]},
{name:"Superman Hold",muscle:"Lower Back/Glutes",avoid:["Lower Back"]}
```

- [ ] **Step 4: Verify in browser**

Generate and shuffle several times. Core slot should show variety.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: expand core exercise pool"
```

---

## Task 6: Expand EX.hiit pools

**File:** `index.html` — find `EX.hiit=` (defined after `EX.strength`)

- [ ] **Step 1: Add exercises to hiit.upper_push**

Append to each tier of `EX.hiit.upper_push`:
```js
// full
{name:"Dumbbell Push Press",muscle:"Shoulders/Triceps",avoid:["Shoulders","Wrists"]},
{name:"Plyometric Push-Up",muscle:"Chest/Shoulders",avoid:["Shoulders","Wrists"]}

// home — same additions
{name:"Dumbbell Push Press",muscle:"Shoulders/Triceps",avoid:["Shoulders","Wrists"]},
{name:"Plyometric Push-Up",muscle:"Chest/Shoulders",avoid:["Shoulders","Wrists"]}

// minimal
{name:"Plyometric Push-Up",muscle:"Chest/Shoulders",avoid:["Shoulders","Wrists"]},
{name:"Band Squat to Press",muscle:"Full Body",avoid:["Shoulders","Knees"]}
```
Before adding `Band Squat to Press` to minimal: search `index.html` for `Band Squat to Press` — if it already exists in that tier, skip it to avoid a duplicate.

- [ ] **Step 2: Add exercises to hiit.upper_pull**

Append to `full` and `home` tiers:
```js
{name:"Renegade Row",muscle:"Back/Core",avoid:["Wrists","Shoulders","Lower Back"]},
{name:"Dumbbell High Pull",muscle:"Back/Shoulders",avoid:["Shoulders","Wrists"]}
```

- [ ] **Step 3: Add exercises to hiit.lower_push**

Append to `full` and `home` tiers:
```js
{name:"Broad Jump",muscle:"Quads/Glutes",avoid:["Knees","Ankles"]},
{name:"Lateral Box Jump",muscle:"Quads/Glutes/Hips",avoid:["Knees","Ankles","Hips"]}
```
Append to `minimal`:
```js
{name:"Broad Jump",muscle:"Quads/Glutes",avoid:["Knees","Ankles"]},
{name:"Lateral Shuffle",muscle:"Quads/Glutes/Hips",avoid:["Knees","Ankles","Hips"]}
```

- [ ] **Step 4: Add exercises to hiit.lower_pull**

Append to `full`:
```js
{name:"Sumo Deadlift High Pull",muscle:"Full Body/Back",avoid:["Lower Back","Wrists","Shoulders"]}
```
Append to `home` and `minimal`:
```js
{name:"Dumbbell Swing",muscle:"Glutes/Back",avoid:["Lower Back","Wrists"]}
```

- [ ] **Step 5: Add exercises to hiit.core**

Append to all tiers:
```js
{name:"V-Up",muscle:"Core/Abs",avoid:["Lower Back"]},
{name:"Tuck Jump",muscle:"Core/Cardio",avoid:["Knees","Ankles"]}
```

- [ ] **Step 6: Verify in browser**

Switch to HIIT style. Generate and shuffle. Confirm new exercises appear.

- [ ] **Step 7: Commit**
```bash
git add index.html
git commit -m "feat: expand hiit exercise pools"
```

---

## Task 7: Add DEMO entries for new exercises

**File:** `index.html` — find the `DEMO` object (the large object starting with `"Barbell Bench Press":{...}`)

- [ ] **Step 1: Add DEMO entries**

Add these entries to the `DEMO` object. Each key must match the exercise `name` exactly (case-sensitive):

```js
"Close Grip Bench Press":{desc:"Narrow grip bench press targeting triceps and inner chest",cues:["Hands shoulder-width or closer","Lower bar to mid-chest","Elbows stay tucked in","Press explosively"]},
"Weighted Dip":{desc:"Parallel bar dip with added weight for chest and triceps",cues:["Lean forward slightly for chest emphasis","Lower until elbows hit 90 degrees","Keep elbows from flaring","Drive up through palms"]},
"Cable Lateral Raise":{desc:"Raise cable handle out to the side to target the lateral delt",cues:["Start with handle in front of body","Raise to shoulder height","Slight bend in elbow","Lower slowly"]},
"Pec Deck":{desc:"Machine fly isolating the chest",cues:["Adjust seat so handles are at chest height","Keep elbows at 90 degrees","Squeeze chest hard at center","Open slowly with control"]},
"Tricep Pushdown":{desc:"Cable pushdown isolating the triceps",cues:["Elbows pinned to sides","Push bar to full extension","Squeeze triceps at bottom","Return slowly"]},
"Dumbbell Chest Fly":{desc:"Lying dumbbell fly opening across the chest",cues:["Lie on bench, dumbbells above chest","Lower in wide arc to chest level","Keep soft bend in elbows","Squeeze chest to return"]},
"Dumbbell Lateral Raise":{desc:"Raise dumbbells to sides for lateral delt isolation",cues:["Slight forward torso lean","Lead with elbows","Raise to shoulder height","Lower with control"]},
"Diamond Push-Up":{desc:"Push-up with hands close together forming a diamond shape",cues:["Thumbs and forefingers touch","Elbows track back not out","Lower chest to hands","Press fully to lockout"]},
"Pike Push-Up":{desc:"Hips high push-up targeting the shoulders",cues:["Form an inverted V with hips high","Lower head toward floor","Elbows go back and out","Press back to start"]},
"Band Lateral Raise":{desc:"Lateral raise against band resistance",cues:["Stand on band center","Hold ends at sides","Raise arms to shoulder height","Control the return"]},
"Band Overhead Press":{desc:"Press band overhead from shoulder level",cues:["Stand on band","Hold at shoulders","Press straight overhead","Lower controlled"]},
"T-Bar Row":{desc:"Row a barbell anchored in a landmine or T-bar apparatus",cues:["Hinge to 45 degrees","Grip the handles narrow","Pull to lower chest","Squeeze shoulder blades hard"]},
"Chest-Supported Row":{desc:"Row on an incline bench to eliminate lower back involvement",cues:["Chest flat on incline pad","Let arms hang fully","Pull dumbbells/bar to hips","Squeeze back at top"]},
"Straight Arm Pulldown":{desc:"Pull cable down in an arc with nearly straight arms",cues:["Arms mostly straight","Start high, pull to hips","Feel lats stretch at top","Control the return"]},
"Hammer Curl":{desc:"Curl with a neutral (hammer) grip to hit biceps and brachialis",cues:["Palms face each other","Keep elbows pinned","Curl to shoulder","Lower slowly"]},
"Preacher Curl":{desc:"Curl on a preacher bench for strict bicep isolation",cues:["Upper arm flat on pad","Full stretch at bottom","Curl to chin level","Lower completely"]},
"Cable Bicep Curl":{desc:"Curl a cable bar to isolate the biceps with constant tension",cues:["Elbows pinned to sides","Curl to chin","Hold 1 second at top","Lower fully"]},
"Dumbbell Hammer Curl":{desc:"Hammer grip curl with dumbbells",cues:["Palms face in throughout","Keep elbows still","Curl to shoulder height","Lower controlled"]},
"Dumbbell Bicep Curl":{desc:"Standard dumbbell curl for bicep isolation",cues:["Start with full arm extension","Curl up rotating palm","Squeeze at top","Lower slowly"]},
"Chest-Supported Dumbbell Row":{desc:"Row with chest on incline bench for strict back isolation",cues:["Chest flat on incline","Arms hang fully down","Pull elbows past torso","Squeeze at top"]},
"Dumbbell Rear Delt Fly":{desc:"Bent-over fly targeting the rear deltoid",cues:["Hinge forward to parallel","Soft bend in elbows","Raise out and back","Pinch shoulder blades"]},
"Band Bicep Curl":{desc:"Curl a resistance band for bicep work",cues:["Stand on band","Hold ends palms up","Curl to shoulders","Lower slowly"]},
"Band Face Pull":{desc:"Pull band to face level for rear delt and upper back",cues:["Anchor band at face height","Pull to temples","Rotate hands outward","Hold 1 second"]},
"Leg Extension":{desc:"Machine exercise isolating the quadriceps",cues:["Adjust pad just above ankle","Press to full extension","Squeeze quads at top","Lower slowly"]},
"Front Squat":{desc:"Barbell held in front rack position, squat to depth",cues:["Bar rests on front delts","Elbows high throughout","Torso stays upright","Squat to parallel or below"]},
"Smith Machine Squat":{desc:"Squat on a fixed-track Smith machine",cues:["Feet slightly forward","Squat to parallel","Drive through heels","Keep back flat"]},
"Sissy Squat":{desc:"Knees travel far forward while heels stay elevated for quad isolation",cues:["Hold support with one hand","Rise on toes","Lower knees forward toward floor","Keep torso and thighs in line"]},
"Box Step-Up":{desc:"Step up onto a box one leg at a time",cues:["Full foot on box","Drive through heel","Stand fully at top","Step down controlled"]},
"Dumbbell Sumo Squat":{desc:"Wide stance squat holding a dumbbell between legs",cues:["Wide stance, toes angled out","Hold dumbbell low centered","Sit straight down","Squeeze glutes at top"]},
"Reverse Lunge":{desc:"Step backward into a lunge to reduce knee stress",cues:["Step straight back","Lower rear knee to floor","Keep front shin vertical","Drive through front heel to return"]},
"Wall Sit":{desc:"Hold a static squat position against the wall",cues:["Back flat on wall","Thighs parallel to floor","Feet flat","Hold and breathe steadily"]},
"Single Leg Squat":{desc:"Pistol squat or assisted single-leg squat",cues:["Stand on one foot","Extend other leg forward","Lower as deep as possible","Drive up through standing heel"]},
"Band Lateral Walk":{desc:"Walk sideways against band resistance for glute med activation",cues:["Band around ankles or knees","Slight squat position throughout","Step laterally against resistance","Keep toes forward"]},
"Good Morning":{desc:"Hinge forward with barbell on upper back to load hamstrings",cues:["Bar on upper back like a squat","Soft bend in knees","Hinge until torso near parallel","Drive hips forward to stand"]},
"Nordic Curl":{desc:"Eccentric hamstring exercise lowering from kneeling position",cues:["Feet anchored","Lower your body slowly with hamstrings","Only lower as far as you can control","Use hands to assist return"]},
"Seated Leg Curl":{desc:"Machine curl in the seated position for hamstring isolation",cues:["Pad just above ankle","Curl toward glutes","Squeeze hamstrings at end range","Lower slowly"]},
"Glute Ham Raise":{desc:"Full range hamstring and glute exercise on GHD machine",cues:["Feet anchored on pads","Lower body under control","Use hamstrings to pull back up","Keep torso straight"]},
"Cable Kickback":{desc:"Kick cable attachment back to isolate the glute",cues:["Ankle strap on working leg","Slight forward lean","Kick leg straight back","Squeeze glute at top"]},
"Donkey Kick":{desc:"On all fours, kick one leg back and up",cues:["On hands and knees","Keep hips level","Drive heel toward ceiling","Squeeze glute at top"]},
"Fire Hydrant":{desc:"On all fours, raise knee out to the side for hip abduction",cues:["On hands and knees","Keep core tight","Raise knee out to side","Hold 1 second at top"]},
"Dumbbell Good Morning":{desc:"Good morning variation holding a dumbbell at chest",cues:["Hold dumbbell at chest","Soft bend in knees","Hinge forward at hips","Drive hips forward to stand"]},
"Curtsy Lunge":{desc:"Lunge crossing the back leg behind for glute and adductor focus",cues:["Cross back leg behind front","Lower rear knee toward floor","Keep chest tall","Drive up through front heel"]},
"Band Kickback":{desc:"Glute kickback using a resistance band",cues:["Anchor band low","Loop around ankle","Kick leg back and up","Squeeze glute at top"]},
"Band Hip Thrust":{desc:"Hip thrust with band resistance across hips",cues:["Band anchored in front","Across hip crease","Drive hips up against band","Squeeze glutes at top"]},
"Cable Crunch":{desc:"Kneel under cable and crunch down for weighted ab isolation",cues:["Kneel facing stack","Hold rope at sides of head","Crunch elbows to thighs","Return slowly"]},
"Decline Sit-Up":{desc:"Sit-up on a decline bench for increased range of motion",cues:["Feet anchored on decline","Cross arms or hold weight","Lower fully to bench","Crunch up explosively"]},
"Ab Wheel Rollout":{desc:"Roll the ab wheel out from standing or kneeling",cues:["Start kneeling","Roll out slowly and far","Keep lower back flat","Pull back using core"]},
"Landmine Rotation":{desc:"Rotate barbell in landmine attachment across the body",cues:["Hold bar with both hands","Rotate from hips not arms","Keep core braced","Return under control"]},
"Captain's Chair Leg Raise":{desc:"Hanging from captain's chair, raise knees or legs",cues:["Elbows on pads","Lower back against pad","Raise knees or legs to 90","Lower slowly"]},
"Hollow Body Hold":{desc:"Lie flat and hold a compressed hollow position",cues:["Press lower back to floor","Arms overhead, legs extended","Raise both slightly off ground","Breathe and hold"]},
"Russian Twist":{desc:"Seated rotation with weight to work the obliques",cues:["Lean back 45 degrees","Feet off floor or on floor","Rotate side to side","Control the movement"]},
"Bicycle Crunch":{desc:"Alternating elbow to knee crunch targeting obliques",cues:["Hands lightly behind head","Alternate elbow to opposite knee","Fully extend the other leg","Keep lower back flat"]},
"Superman Hold":{desc:"Lie face down and raise arms and legs off the floor",cues:["Lie flat face down","Raise arms and legs simultaneously","Hold 2 seconds at top","Lower slowly"]},
"Dumbbell Push Press":{desc:"Use leg drive to press dumbbells overhead",cues:["Slight dip in knees","Explode up with legs","Press dumbbells overhead","Lower controlled"]},
"Plyometric Push-Up":{desc:"Explosive push-up where hands leave the floor",cues:["Standard push-up position","Push up explosively","Hands leave floor at top","Land with soft elbows"]},
"Renegade Row":{desc:"Plank position, row one dumbbell at a time",cues:["Wide stance in push-up position","Row one dumbbell to hip","Keep hips level","Alternate sides"]},
"Dumbbell High Pull":{desc:"Explosive upright row with dumbbells",cues:["Start at hips","Drive elbows high and fast","Pull to chin level","Lower controlled"]},
"Broad Jump":{desc:"Two-foot explosive jump forward for power",cues:["Feet hip width","Swing arms back","Explode forward and up","Land with soft knees"]},
"Lateral Box Jump":{desc:"Jump sideways onto a box",cues:["Stand beside box","Jump laterally onto box","Land softly both feet","Step down and repeat"]},
"Lateral Shuffle":{desc:"Rapid lateral steps for conditioning",cues:["Athletic stance","Shuffle rapidly side to side","Stay low throughout","Keep feet from crossing"]},
"Sumo Deadlift High Pull":{desc:"Sumo deadlift combined with an explosive upright row",cues:["Wide stance sumo setup","Drive hips explosively","Pull bar to chin","Lower bar to floor"]},
"Dumbbell Swing":{desc:"Dumbbell swing mimicking a kettlebell swing",cues:["Hinge at hips","Swing dumbbell between legs","Snap hips forward","Let arms float up naturally"]},
"V-Up":{desc:"Simultaneously raise legs and torso to form a V shape",cues:["Lie flat, arms overhead","Raise legs and upper body together","Touch toes or reach toward feet","Lower slowly"]},
"Tuck Jump":{desc:"Jump and pull knees to chest at peak",cues:["Jump explosively","Tuck knees to chest at top","Land softly","Reset immediately"]}
```

- [ ] **Step 2: Verify a new exercise's demo modal**

In the browser, generate a workout. If a new exercise appears (e.g. "Hammer Curl"), tap it. The demo modal should show the description and form cues. If it doesn't appear in the workout, manually check by temporarily setting `exSeeds` to force it, or just reshuffle.

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "feat: add DEMO entries for new exercises"
```

---

## Task 8: Add VIDEO_MAP entries for new exercises

**File:** `index.html` — find the `VIDEO_MAP` object (around line 36)

Note: Missing VIDEO_MAP entries are gracefully handled — the app hides the video thumbnail area. Add what you can; leave the rest for later. Verify all IDs before committing by checking `https://www.youtube.com/watch?v=<ID>`.

- [ ] **Step 1: Add VIDEO_MAP entries**

Add these to the `VIDEO_MAP` object:
```js
"Close Grip Bench Press":"nG6kB7fGdtI",
"Weighted Dip":"2z8JmcrW-As",
"Cable Lateral Raise":"3VcKaXpzqRo",
"Pec Deck":"Z57CtFmRMxA",
"Tricep Pushdown":"2-LAMcpzODU",
"Dumbbell Chest Fly":"eozdVDA78K0",
"Dumbbell Lateral Raise":"kDqklk1ZESo",
"Diamond Push-Up":"J0DXBSpghaE",
"T-Bar Row":"j3x7tYnLAd4",
"Hammer Curl":"zC3nLlEvin4",
"Preacher Curl":"fIWP-FRFNU0",
"Dumbbell Hammer Curl":"TwD-YGVP4Bk",
"Leg Extension":"YyvSfVjQeL0",
"Front Squat":"uYumuL_G_V0",
"Nordic Curl":"d8_UEKiZzw4",
"Seated Leg Curl":"1Tq3QdYUuHs",
"Cable Crunch":"kiuVA0gs3EI",
"Decline Sit-Up":"1fbU_MkV7NE",
"Hollow Body Hold":"LlDNef_Unt4",
"Bicycle Crunch":"1we3bh9uhqY",
"Donkey Kick":"SJ3xR-KXNTI",
"Reverse Lunge":"D7KaRcUTQeE",
"Russian Twist":"wkD8rjkodUI",
"V-Up":"7UVgs18Y1P4",
"Renegade Row":"Xy-OIqhCEcA",
"Broad Jump":"ySXVNKFmKL0",
"Dumbbell Push Press":"J9AC8BohFRE"
```

- [ ] **Step 2: Spot-check two video links**

For two of the new entries, open the YouTube URL `https://www.youtube.com/watch?v=<ID>` in a browser to confirm the ID is correct. Update any wrong IDs.

- [ ] **Step 3: Commit**
```bash
git add index.html
git commit -m "feat: add VIDEO_MAP entries for new exercises"
```

---

## Task 9: Update swap logic — hide button when no alternatives

**File:** `index.html` — the `workout` useMemo (~line 132) and the exercise card render (~line 222)

This task adds `swappable` to the workout object and uses it to conditionally render the swap button.

- [ ] **Step 1: Update the workout useMemo**

Find the existing useMemo (starts with `const workout=useMemo(()=>{`). Replace it entirely with:

```js
const workout=useMemo(()=>{
  const db=EX[style]||EX.strength;const isH=style==="hiit"||style==="crossfit";
  const exercises=baseWorkout.exercises.map((ex,idx)=>{const key=`${curDay}-${idx}`;if(exSeeds[key]!==undefined){const cat=getCatForSlot(dayType,idx);const pool=filt((db[cat]||{})[equipment]||[],aches);if(pool.length>0){const rng2=seededRandom(exSeeds[key]);const picked=pick(pool,1,rng2)[0];if(picked)return{...picked,sets:isH?int.rounds:int.sets,reps:isH?`${int.hiitWork}/${int.hiitRest}`:int.reps,rest:isH?"between rounds":int.rest,isHIIT:isH};}}return ex;});
  const currentNames=new Set(exercises.map(e=>e.name));
  const swappable=exercises.map((ex,idx)=>{const cat=getCatForSlot(dayType,idx);const basePool=filt((db[cat]||{})[equipment]||[],aches);const otherNames=new Set([...currentNames].filter(n=>n!==ex.name));return basePool.some(e=>e.name!==ex.name&&!otherNames.has(e.name));});
  return{warmup:baseWorkout.warmup,exercises,swappable};
},[seed,curDay,exSeeds,equipment,aches,style,intLvl,days]);
```

Key changes from original:
- After building `exercises`, computes `currentNames` (set of all exercise names in the workout)
- For each slot, checks if the category pool has any exercise that is neither the current exercise nor already displayed in another slot
- Returns `swappable` array alongside `exercises` and `warmup`

- [ ] **Step 2: Update the swap button render**

Find the exercise card render block (the `workout.exercises.map` call, around line 222). Find the swap button:
```js
<button onClick={e=>{e.stopPropagation();shuffleSingle(i);}} style={{...}}>↻</button>
```

Replace it with:
```js
{workout.swappable[i]&&<button onClick={e=>{e.stopPropagation();shuffleSingle(i);}} style={{background:"rgba(226,176,111,0.06)",border:"1px solid rgba(226,176,111,0.18)",color:C.gold,width:32,height:32,borderRadius:8,cursor:"pointer",fontSize:14,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0}}>↻</button>}
{!workout.swappable[i]&&<div style={{width:32,height:32,flexShrink:0}}/>}
```

The `<div>` spacer preserves layout alignment when the button is hidden.

- [ ] **Step 3: Verify swap button behavior**

Open in browser. Check the following manually:

1. **Normal case:** Generate a Full Gym workout. Swap buttons (↻) should appear on all exercises. Tap one — the exercise should change. Tap again — should change again (different exercise).

2. **Max avoidance:** On setup screen, check ALL body areas under "Aches or Areas to Avoid". Generate. Most exercises will fall back to Bodyweight Circuit. Swap buttons should be hidden on those slots (no alternatives after filtering). No JS errors in browser console.

3. **Minimal equipment + some avoidance:** Select Minimal equipment, check "Shoulders" and "Knees". Generate. Some exercises may have no alternatives — those swap buttons should be hidden.

4. **Shuffle All:** Tap "↻ Shuffle All". Workout changes. Swap buttons re-evaluate and show/hide correctly for the new workout.

- [ ] **Step 4: Check browser console for errors**

Open DevTools (F12) → Console. Generate several workouts, shuffle, swap exercises. There should be zero errors.

- [ ] **Step 5: Commit**
```bash
git add index.html
git commit -m "feat: add swappable flag, hide swap button when no alternatives"
```

---

## Task 10: Final verification

- [ ] **Step 1: Full shuffle diversity test**

Select Full Gym + Strength + 4 days. Generate. Tap "↻ Shuffle All" 10 times in a row. Confirm you see different exercises most of the time — the expanded pool should mean you rarely see the same combination twice.

- [ ] **Step 2: Cross-style test**

Repeat with HIIT style and CrossFit style. Both should show the expanded pools. (CrossFit aliases HIIT, so changes from Task 6 apply to both.)

- [ ] **Step 3: All done**

All changes were committed in Tasks 1–9. No new commit needed here — this is a verification-only step.
