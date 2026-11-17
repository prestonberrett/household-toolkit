---
name: meal-planner
description: Generate a dated dinner plan for the rest of the month (or any date range) from the user's weekday theme rotation and food preferences. Use when asked to plan dinners, make a meal plan, or build this month's menu.
---

# Meal Planner

Produce a dated dinner plan where each weekday can have a fixed theme and the recipe within the theme rotates. All personal choices live in `preferences.md`, in the same folder as this file. This skill holds only the method and the output format.

## Step 1: Load preferences

Read `preferences.md` from the folder that contains this `SKILL.md`.

- **If it exists and has settings in it**, use it as is. Do not re-ask questions it already answers.
- **If it is missing, empty, or contains only a comment**, run first-time setup (below), write the file, then continue to Step 2.
- **If the user says "update my preferences"** (or changes a preference mid-chat and wants it kept), edit `preferences.md` with the change, then continue.

### First-time setup

Ask these in one message (use a multiple-choice question tool if one is available), then save the answers:

1. **Themes:** Which theme goes with each weekday, or no themes at all?
2. **Exclusions:** Any foods to exclude (allergies, dislikes, dietary rules)? Record each one separately.
3. **Recipes:** Do you already have recipes?
   - **Yes:** ask where (a file, a folder, an app export such as Paprika, or pasted text). Record the location in `preferences.md`. If the recipes are pasted or in an export, offer to save them into a recipe file in the format below.
   - **No, or only a few:** offer to generate a starter list of dishes for each theme, filtered by the exclusions, and save it to a recipe file.
4. **Plan location:** Which folder should finished plans (and any recipe file you create) be saved in? Save the answer as an absolute path. If that folder is inside a git repository, tell the user and suggest adding it to `.gitignore`, since plans and recipes are personal.
5. **Planning horizon:** How far ahead by default (rest of this month, next 4 weeks, ...)?
6. **Household size:** How many people are you cooking for?
7. **Repeat window:** How long before a dish can repeat (default: 3 weeks)?

Write the answers to `preferences.md` in the folder that contains this `SKILL.md`, in the format shown in `preferences.example.md` (same folder), replacing any placeholder comment. Then read the file back to confirm it was saved. If the write fails, say so and show the user the contents to save themselves. Do not say preferences were saved unless the read-back confirms it.

Setup happens once. After that, only return to recipes when a theme's pool runs out (Step 3).

### Recipe file format

Recipe files use one heading per theme and one bullet per dish:

```
## Taco Tuesday
- Ground beef tacos
- Chicken tinga tacos
```

Use this format whenever the skill creates or adds to a recipe file, so other tools can read and extend the same files.

## Step 2: Gather inputs

1. **Today's date:** get it from the environment (for example, run `date`). Never assume it.
2. **Date range:** start **today** and run to the end of the default horizon from preferences, unless the user names a range. Map dates to weekdays from the real calendar. Skip any dates the user names (holidays, eating out) and note them.
3. **Recipe pool per theme:** read the recipe sources listed in preferences, plus the most recent previous `Meal Plan - ....md` in the plan location if one exists, to continue the rotation and avoid repeats.
4. If a listed source is unavailable, say so in one line and work with what you have. Do not invent file contents.

## Step 3: Build the plan

1. List every date in the range with its weekday and theme.
2. For each theme, take the eligible dishes: everything in the pool minus anything used within the repeat window.
3. Assign dishes so each theme cycles through its pool before repeating.
4. Label each assigned dish with its main protein (chicken, beef, pork, vegetarian, ...).
5. **Protein check.** Scan the plan in date order. If the same main protein appears on consecutive planned days, fix it by swapping in another eligible dish within the same theme (swap across weeks if needed), then scan again. If the pool does not allow it, keep the plan but say so in the Notes and suggest which theme needs more variety.
6. If a theme has no unused dish left, propose one new recipe that fits the theme and the exclusions. Mark it `(new)` and offer to add it to the matching recipe file.
7. Re-check every dish, including sauces and toppings, against the exclusions.

## Step 4: Output

```
# Meal Plan - [Month Year]

| Date | Recipe |
|------|--------|
| Nov 17 | Ground Beef Tacos |
| ... | ... |

## Notes
- Rotation: recipes rotate within each theme to avoid repeats.
- Constraints: [every exclusion from preferences.md, listed in full; "none" if there are none]
- Swaps: [swaps, skipped dates, (new) recipes, protein runs that could not be avoided, or "none"]
```

- Title is `Meal Plan - [Month Year]`, or `Meal Plan - Rest of [Month Year]` when starting mid-month.
- The table has exactly two columns, `Date | Recipe`, one row per day, dates as `Mon D`.
- Keep notes short.

## Swaps and saving

- To swap a dish, replace it with another from the same theme, update the table, and note the swap.
- Save each plan as `Meal Plan - [Month Year].md` in the plan location from preferences, so the next month can read it for rotation history.
