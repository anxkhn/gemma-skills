---
name: tone-song
description: Make a short playable song with Tone.js.
metadata:
  homepage: https://github.com/anxkhn/gemma-skills/tree/main/tone-song-skill
---

# Tone song

## Instructions

Use this skill when the user asks to make, compose, generate, or play a short song, melody, beat, loop, or musical idea.

Call the `run_js` tool with the following exact parameters:

- script name: `index.html`
- data: A JSON string with the following fields:
  - prompt: String. The user's original music request.
  - title: String. Optional short title.
  - bpm: Number. Optional tempo from 60 to 160.
  - melody: Array of note events. Required. Each event has:
    - time: String in Tone.js transport time, such as `0:0:0`, `0:1:0`, or `1:2:2`.
    - note: String pitch-octave notation, such as `C4`, `F#4`, or `Bb5`.
    - duration: String Tone.js duration, such as `8n`, `4n`, `2n`, or `1m`.
  - chords: Array of chord events. Optional. Each event has:
    - time: String in Tone.js transport time.
    - notes: Array of pitch-octave strings.
    - duration: String Tone.js duration.
  - bass: Array of bass events. Optional. Same shape as melody events.
  - drums: Array of drum events. Optional. Each event has:
    - time: String in Tone.js transport time.
    - type: String, one of `kick`, `snare`, or `hat`.

The payload is the song. Do not send only a prompt, mood, or song name. If the user asks for a recognizable melody, generate the melody as `melody` note events. If you cannot confidently encode the exact tune, create a short original melody that matches the request and make the title clear.

Do not use `run_intent`. Audio playback requires the returned player view, so after the tool returns, tell the user that the song is ready and they can press Play.
