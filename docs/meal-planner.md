# Meal planner: design notes

## Goal

Turn a household's standing dinner habits into a dated plan with almost no effort each month. The user states their habits once. After that, "plan my dinners" is the whole request.

## How a run works

1. Read `preferences.md` (themes, exclusions, plan location, recipe sources, horizon, household size, repeat window).
2. If it is missing, empty, or only a placeholder comment, ask the setup questions, write the file, and continue.
3. Build the pool of eligible dishes for each theme from the recipe sources, minus anything used inside the repeat window.
4. Assign dishes day by day, scan for the same main protein on consecutive days and swap within the theme to fix it, then re-check every dish against the exclusions.
5. Output a `Date | Recipe` table with short notes, and save it as `Meal Plan - [Month Year].md` in the plan location so the next run can read it for rotation history.

## Decisions

**Fixed themes, rotating recipes.** Deciding what to cook every night is tiring. Deciding between a handful of dishes inside a known theme is not. Fixing the theme per weekday removes most of the choice, and rotating the dish inside it keeps the week from feeling repetitive.

**Preferences live in `preferences.md`, not `SKILL.md`.** The skill stays generic and shareable. Personal choices are data, not instructions, so they are stored separately and changed without touching the skill. A skill that rewrites its own instructions drifts over time and mixes one household's choices into shared code.

**Setup happens once.** The skill asks its questions only when there is no usable `preferences.md`. After that it asks nothing unless a theme runs out of unused dishes, or the user says "update my preferences."

**Existing recipes come first.** Setup asks whether the user already has recipes before offering to generate any. People with a library should never be pushed into a search or a starter list.

**Personal data stays out of the repo.** Plans and recipes are saved to a folder the user chooses at setup, not wherever Claude happens to be running. If that folder is inside a git repository, the skill says so and suggests a `.gitignore` entry. The repo itself ignores `meals/`, saved plans and `preferences.md`.

**Preferences are verified.** After writing `preferences.md` the skill reads it back, so it never reports a save that did not happen.

**A shared recipe file format.** Recipe files use one heading per theme and one bullet per dish. Any other tool, such as a future recipe-finder agent, can then add dishes to the same files without changes to the skill.

**A skill, not an agent.** The task follows a fixed procedure with known inputs and a known output. It does not need to search, browse or decide what to do next. An agent fits better for open-ended recipe discovery, which is planned separately (see the roadmap in the README).

## Files

| File | Purpose |
|------|---------|
| `skills/meal-planner/SKILL.md` | Method and output format. Generic. |
| `skills/meal-planner/preferences.md` | One household's settings. Written by the skill on first run. Ignored by git. |
| `skills/meal-planner/preferences.example.md` | Sample of a completed preferences file, used as the format reference during setup. |
| `examples/meal-planner.md` | Sample plan output. |

## Known limits

- Dishes are checked against exclusions by name and by what the model knows of the recipe. Hidden ingredients in a pasted recipe can slip through, so allergies deserve a final look by the cook.
- Rotation history comes from earlier saved plans. If none are available, the first month cannot avoid repeating last month's dishes.
- Household size is recorded but does not yet change which dishes are chosen.
