# SACS Manifest: Omnichord Module Index

| Module | Status | Owns (paths) | Key responsibilities | Doc |
|---|---:|---|---|---|
| main | complete | `src/omnichord/main.py` | CLI entrypoint, 60 FPS game loop, component orchestration | [modules/main.md](modules/main.md) |
| input | complete | `src/omnichord/input/` | Keyboard input via evdev, chord/strum detection, N-key rollover | [modules/input.md](modules/input.md) |
| music | complete | `src/omnichord/music/` | Chord theory, frequency calculations, strum plate mechanics | [modules/music.md](modules/music.md) |
| audio | complete | `src/omnichord/audio/` | Real-time synthesis, voice management, ADSR envelopes, waveform generation | [modules/audio.md](modules/audio.md) |
| display | complete | `src/omnichord/display/` | Curses-based ASCII terminal UI rendering | [modules/display.md](modules/display.md) |
