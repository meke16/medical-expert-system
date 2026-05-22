# AI-Powered Medical Decision Support System

## What this system is

This project is an AI-style medical decision support system. It is designed to simulate a simple differential diagnosis flow by comparing a user's reported symptoms against a static medical knowledge base, then ranking the most likely conditions.

It is intended for educational and demonstration purposes only. It does not replace a doctor, a clinic, or any form of professional diagnosis.

## How the system works overall

The system has two main parts:

1. The frontend collects age, gender, language, and selected symptoms.
2. The FastAPI backend applies the inference logic and returns ranked possible conditions.

The data flow is simple:

- The diagnosis screen first asks the backend for the full symptom list from `GET /api/symptoms`.
- The frontend filters that list locally while the user types in the search box.
- When the user submits, the frontend sends the selected symptoms, age, gender, and language to `POST /api/diagnose`.
- The backend compares the request against the disease knowledge base and returns up to 5 matches.
- The results and report pages display those ranked matches, explanations, matched symptoms, and treatment guidance.

## What algorithm it uses

This codebase does not use machine learning. It uses a weighted, rule-based scoring algorithm, but it is presented like an AI assistant because it gives ranked guidance from user input.

The scoring logic lives in `backend/engine/diagnosis.py` and works like this:

1. Load all diseases from `backend/data/diseases.json` when the backend starts.
2. Build the full symptom list from the union of every disease's symptom list.
3. For each disease, check which of the user's symptoms are present in that disease profile.
4. Ignore diseases that do not match at least one symptom.
5. Compute a score using two parts:
   - symptom overlap: how many of the selected symptoms appear in the disease profile
   - symptom coverage: how much of the disease profile is covered by the selected symptoms
6. Combine those two parts with weights:
   - 70% symptom overlap
   - 30% symptom coverage
7. Apply demographic modifiers:
   - age group: child, adult, senior
   - gender risk multiplier: male, female, other fallback
8. Cap the final confidence below 100%.
9. Sort all matching diseases by confidence and return the top 5.

In short, the engine is a symptom-matching ranker, not a predictive AI model.

## Exact confidence idea

The core idea is:

```text
raw_confidence = (0.7 * symptom_overlap) + (0.3 * symptom_coverage)
final_confidence = raw_confidence * age_modifier * gender_modifier
```

The backend then converts that to a percentage and returns it in the response.

## How the static data works

The medical knowledge base is stored in `backend/data/diseases.json`.

Each disease entry contains:

- a unique `id`
- an English name and localized names for supported languages
- a severity level
- a list of associated symptoms
- an explanation string
- a treatment string
- age-based risk multipliers
- gender-based risk multipliers

That JSON file is loaded into memory when `backend/engine/diagnosis.py` is imported. After that, every request uses the in-memory data. There is no database, no ORM, and no persistence layer for patient records.

## Localization behavior

The system supports language-aware output for `en`, `am`, and `om`.

When the frontend asks for another language, the backend tries to return the localized disease name, explanation, and treatment text. If a localized field is missing, the engine falls back to the English version.

The symptom list in the UI is also translated on the frontend through the language context.

## Frontend search behavior

The symptom search box is not a semantic search engine. It is a simple text filter over the symptom labels shown in the UI.

That means:

- typing "fev" will match fever
- typing "pain" will match symptoms whose display label contains pain
- it does not understand medical synonyms or misspellings beyond the exact text match behavior built into the browser UI

## Backend endpoints

- `GET /health` returns a simple health status.
- `GET /api/symptoms` returns the complete symptom list.
- `POST /api/diagnose` runs the rule-based inference engine.
- `POST /api/report` builds a printable report payload from diagnosis results.
- `GET /api/about` returns the system overview and FAQ data.

## Common questions

### Is this AI?

Not in the machine-learning sense. It is an AI-style deterministic rule-based expert system.

### Why does age matter?

Some conditions are more common or more dangerous in children or older adults, so the engine applies age-group modifiers to the confidence score.

### Why does gender matter?

Some conditions have slightly different risk patterns across genders, so the engine applies a gender-based multiplier.

### Why are some results marked urgent or critical?

The severity label comes directly from the disease record in the JSON knowledge base.

### Why do I sometimes get no results?

If the selected symptoms do not overlap with any disease profile, the engine returns an empty list.

### Is my data stored?

No persistent storage is used. The request is processed in memory and discarded after the response is returned.

## Important limitation

This system can suggest likely conditions, but it cannot confirm a diagnosis. Its output is only a ranked hint based on the symptom list and the static rules in the data file.

If the symptoms are severe, unusual, or worsening, the correct next step is medical care, not another search in the app.