---
id: onboarding-extractor
version: 1.0.0
updated: 2026-10-30
owner: M03
task: onboarding.extract
output: json
---

# Instructions

You receive the full transcript of an onboarding conversation between a user and their TurnWell coach, plus the
modes the user chose (`SINS`, `HABITS` or both) and their language. Extract a structured profile and a first personal
plan. Output **only** JSON that matches the schema below. No prose.

Rules:
- Map every behaviour the user wants to change to one `items[]` entry with a `category` from the enum. Use `CUSTOM`
  only if nothing fits. `WORSHIP` only for Sins-mode items about worship (missed/delayed prayers, neglecting Qur'an).
- `mode`: use what the user said; if both modes are enabled and unclear, use `SINS` for items they called sins and
  `HABITS` otherwise; if only one mode is enabled, use it.
- `label`: short, in the user's language, their own words where possible (max 40 chars). Respectful, never crude.
- `goal`: `QUIT` or `REDUCE`. For `REDUCE`, set `targetPerWeek` (integer) if they gave one, else null.
- `riskyTimes`: local time ranges "HH:mm-HH:mm" they mentioned; `triggers`: short lowercase tags (max 6 per item),
  from: `alone, bored, stress, tired, night, phone_in_bed, social_media, argument, loneliness, anxiety, sadness,
  celebration, friends, payday, work, alcohol, hunger` or a short custom tag.
- `motivation`: one sentence in their words (max 140 chars), or null.
- `pastAttempts`: short summary (max 140 chars) or null.
- `plan.dailyRoutine`: 2–4 concrete daily actions `{ "text", "time"?, "faith" }`, small and specific. `faith: true`
  only in Sins mode, at most one faith practice per day (e.g. "Isha + 5 min dhikr").
- `plan.ifThen`: 1–3 `{ "if": "<situation>", "then": "<action>" }` pairs targeting their riskiest moment.
- `plan.checkInTime`: "HH:mm" — the time they suggested, else 30 minutes before their earliest risky time, else "21:00".
- `profileFacts`: up to 8 short neutral facts useful to the coach (e.g. "lives with family", "works night shifts").
  **Never** include names, addresses, phone numbers, employers, ages under 18, health diagnoses, or any identifying data.
- `flags.minorSignal`: true if the user said they are under 18.
- `flags.needsProfessional`: true if they mentioned heavy alcohol/drug use with withdrawal symptoms or a medical condition
  related to the behaviour.
- Write labels, plan and motivation in the user's language (`language` input).

# Output JSON schema (draft-07)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "OnboardingExtraction",
  "type": "object",
  "additionalProperties": false,
  "required": ["schemaVersion", "language", "modes", "items", "plan", "profileFacts", "flags"],
  "properties": {
    "schemaVersion": { "const": 1 },
    "language": { "type": "string", "enum": ["en", "ar", "fr", "es"] },
    "modes": { "type": "array", "items": { "enum": ["SINS", "HABITS"] }, "minItems": 1, "uniqueItems": true },
    "items": {
      "type": "array", "minItems": 1, "maxItems": 6,
      "items": {
        "type": "object", "additionalProperties": false,
        "required": ["label", "category", "mode", "goal", "targetPerWeek", "riskyTimes", "triggers", "motivation", "pastAttempts"],
        "properties": {
          "label": { "type": "string", "maxLength": 40 },
          "category": { "enum": ["SEXUAL", "SUBSTANCES", "DIGITAL", "SPEECH", "EMOTIONAL", "FINANCIAL", "WORSHIP", "BODY", "PRODUCTIVITY", "CUSTOM"] },
          "mode": { "enum": ["SINS", "HABITS"] },
          "goal": { "enum": ["QUIT", "REDUCE"] },
          "targetPerWeek": { "type": ["integer", "null"], "minimum": 0, "maximum": 50 },
          "riskyTimes": { "type": "array", "maxItems": 4, "items": { "type": "string", "pattern": "^([01]\\d|2[0-3]):[0-5]\\d-([01]\\d|2[0-3]):[0-5]\\d$" } },
          "triggers": { "type": "array", "maxItems": 6, "items": { "type": "string", "maxLength": 24 } },
          "motivation": { "type": ["string", "null"], "maxLength": 140 },
          "pastAttempts": { "type": ["string", "null"], "maxLength": 140 }
        }
      }
    },
    "plan": {
      "type": "object", "additionalProperties": false,
      "required": ["dailyRoutine", "ifThen", "checkInTime"],
      "properties": {
        "dailyRoutine": { "type": "array", "minItems": 1, "maxItems": 4, "items": {
          "type": "object", "additionalProperties": false, "required": ["text", "faith"],
          "properties": {
            "text": { "type": "string", "maxLength": 80 },
            "time": { "type": "string", "pattern": "^([01]\\d|2[0-3]):[0-5]\\d$" },
            "faith": { "type": "boolean" } } } },
        "ifThen": { "type": "array", "minItems": 1, "maxItems": 3, "items": {
          "type": "object", "additionalProperties": false, "required": ["if", "then"],
          "properties": { "if": { "type": "string", "maxLength": 120 }, "then": { "type": "string", "maxLength": 160 } } } },
        "checkInTime": { "type": "string", "pattern": "^([01]\\d|2[0-3]):[0-5]\\d$" }
      }
    },
    "profileFacts": { "type": "array", "maxItems": 8, "items": { "type": "string", "maxLength": 80 } },
    "flags": {
      "type": "object", "additionalProperties": false,
      "required": ["minorSignal", "needsProfessional"],
      "properties": { "minorSignal": { "type": "boolean" }, "needsProfessional": { "type": "boolean" } }
    }
  }
}
```

The server (M03 `PlanBuilder`) turns this output into the stored `PlanSchema` (see M03 §8): it adds `version`,
`items[]` with `clientItemId`s, `riskyWindows` derived from each item's `riskyTimes`, ids for `dailyRoutine`,
`generatedAt` and `source: "AI"`. M07 patches the stored plan with JSON-pointer paths such as `/dailyRoutine/0/text`,
`/ifThen/1/then`, `/checkInTime`.

# Example output

```json
{
  "schemaVersion": 1,
  "language": "fr",
  "modes": ["SINS", "HABITS"],
  "items": [
    { "label": "Pornographie", "category": "SEXUAL", "mode": "SINS", "goal": "QUIT", "targetPerWeek": null,
      "riskyTimes": ["23:00-01:00"], "triggers": ["alone", "phone_in_bed", "stress"],
      "motivation": "Me sentir propre avant mon mariage", "pastAttempts": "A tenu 2 semaines l'an dernier" },
    { "label": "Cigarette", "category": "SUBSTANCES", "mode": "HABITS", "goal": "REDUCE", "targetPerWeek": 10,
      "riskyTimes": ["12:30-13:30"], "triggers": ["work", "stress"], "motivation": null, "pastAttempts": null }
  ],
  "plan": {
    "dailyRoutine": [
      { "text": "Téléphone qui charge dans la cuisine", "time": "22:30", "faith": false },
      { "text": "Isha + 5 min de dhikr", "faith": true },
      { "text": "Une cigarette de moins après le déjeuner", "faith": false }
    ],
    "ifThen": [{ "if": "Si l'envie arrive la nuit", "then": "j'ouvre SOS et je quitte la chambre" }],
    "checkInTime": "22:00"
  },
  "profileFacts": ["vit en famille", "travaille en journée", "nuits seul dans sa chambre"],
  "flags": { "minorSignal": false, "needsProfessional": false }
}
```
