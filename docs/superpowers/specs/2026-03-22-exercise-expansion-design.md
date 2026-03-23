# Martin Training — Exercise Expansion & Swap Design

**Date:** 2026-03-22
**Status:** Approved

---

## Problem

1. The exercise pool per category/equipment tier is small (~4–6 exercises), causing the shuffle to repeatedly surface the same workouts.
2. The shuffle seed is date-based and deterministic — tapping shuffle on the same day always produces the same workout.
3. Users have no way to swap out an exercise they don't want without regenerating the entire workout.

---

## Goals

1. Expand the exercise library per category by researching exercises from popular fitness programs (GZCLP, PHUL, Reddit PPL, nSuns, 5/3/1, Starting Strength).
2. Allow users to swap individual exercises inline without regenerating the whole workout.
3. Improve shuffle diversity by making manual reshuffles produce distinct results.

---

## Architecture

This is a single static HTML file using CDN React (no build step). All changes are made within `index.html`. The data format and component structure are preserved — no structural refactoring.

---

## Section 1: Exercise Pool Expansion

### Approach
Expand each category from ~4–6 to ~12–15 exercises per equipment tier (`full`, `home`, `minimal`). New entries follow the existing format:

```js
{ name: "Exercise Name", muscle: "Muscle Group", avoid: ["Area1", "Area2"] }
```

### Categories to expand
- `upper_push` — chest, shoulders, triceps
- `upper_pull` — back, biceps, rear delts
- `lower_push` — quads, glutes (push pattern)
- `lower_pull` — hamstrings, glutes (pull pattern)
- `core` — stability and anti-rotation

### Source programs
GZCLP, PHUL, Reddit PPL, nSuns, Wendler 5/3/1, Starting Strength, Arnold Split

### Example additions (full gym tier)
- **upper_push:** Close Grip Bench Press, Weighted Dip, Cable Lateral Raise, Pec Deck, Tricep Pushdown, JM Press
- **upper_pull:** T-Bar Row, Straight Arm Pulldown, Cable Lat Pullover, Hammer Curl, Preacher Curl, Chest-Supported Row
- **lower_push:** Leg Extension, Front Squat, Sissy Squat, Smith Machine Squat, Leg Press (Close Stance)
- **lower_pull:** Good Morning, Nordic Curl, Seated Leg Curl, Glute Ham Raise, Cable Kickback
- **core:** Cable Crunch, Decline Sit-Up, Ab Wheel Rollout, Windmill, Landmine Rotation

Home and minimal tiers receive equivalent dumbbell/band substitutions.

### Demo & video entries
New exercises are added to `DEMO` (form cues) and `VIDEO_MAP` (YouTube IDs) using the same format as existing entries.

---

## Section 2: Exercise Swap UI

### Behavior
- Each exercise card displays a small swap icon (⟳) in the top-right corner.
- Tapping it replaces the exercise **inline** — no modal, no navigation.
- The replacement is randomly selected from the same category pool, filtered by:
  - Same equipment tier
  - User's active avoid list
  - Not the current exercise
  - Not any other exercise currently in the workout
- Sets, reps, and rest remain unchanged — only the exercise name, muscle label, and demo data change.
- If the filtered pool has no valid alternatives, the swap icon is hidden.
- Each tap cycles to a new alternative (uses an incrementing per-exercise swap counter as seed offset).

### State changes
- Add `swapCounts` state: `{ [exerciseIndex]: number }` — increments on each swap tap.
- Swap selection: `pick(filteredPool, 1, seededRandom(baseSeed + swapCounts[i]))`.

---

## Section 3: Shuffle Diversity Fix

### Problem
`seededRandom(dateSeed(date))` is deterministic per day — tapping shuffle repeatedly returns the same workout.

### Fix
Add `shuffleCount` state (integer, starts at 0). Seed becomes `dateSeed(date) + shuffleCount`.

- First load of the day: `shuffleCount = 0` → same behavior as before (stable daily default).
- Each shuffle tap: `shuffleCount++` → new seed, new workout.
- Resets to 0 on new day (date changes naturally).

With the expanded pool (12–15 exercises per category), distinct reshuffles within a day will rarely repeat.

---

## Error Handling

- If a category pool is empty after filtering (all exercises conflict with avoid list), fall back to `Bodyweight Circuit` — same as current behavior.
- If swap pool has no alternatives after filtering, hide the swap icon rather than showing a broken state.

---

## Testing

- Verify all new exercises render correctly in the workout and demo modal.
- Verify swap respects avoid list and equipment tier.
- Verify sequential taps on swap produce different exercises.
- Verify shuffle produces a different workout on each tap.
- Verify the app still works correctly when all body areas are checked (maximum avoidance).
