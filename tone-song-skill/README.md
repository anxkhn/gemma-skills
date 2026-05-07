# Tone Song Skill

A minimal AI Edge Gallery JavaScript skill that creates a short playable Tone.js song from note events.

## Use

Add this skill folder URL in AI Edge Gallery:

```text
https://anxkhn.github.io/gemma-skills/tone-song-skill
```

The skill entry point is:

```text
scripts/index.html
```

It returns an embedded player view. Press **Play** in the player to start audio, because browsers require a user gesture before Web Audio can run.

The skill plays musical notes. It does not synthesize a singing voice or lyrics.

## Input

The model calls `run_js` with:

```json
{
  "prompt": "play a short melody",
  "title": "Quick Melody",
  "bpm": 110,
  "melody": [
    { "time": "0:0:0", "note": "C4", "duration": "8n" },
    { "time": "0:1:0", "note": "E4", "duration": "8n" },
    { "time": "0:2:0", "note": "G4", "duration": "4n" }
  ],
  "chords": [
    { "time": "0:0:0", "notes": ["C4", "E4", "G4"], "duration": "1m" }
  ]
}
```

The skill is payload-driven: `scripts/index.html` does not contain a database of songs. It validates the note events supplied by the LLM and passes them to the Tone.js player.
