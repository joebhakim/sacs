# SACS: Omnichord System Architecture Documentation

## What is SACS?

SACS (System Architecture DAG Set) is a structured documentation system that provides comprehensive, evidence-based architecture documentation for the Omnichord emulator. It uses Mermaid diagrams and code references to help humans and LLMs quickly understand the system's design, data flow, and implementation.

## Purpose

This documentation helps you:
- **Understand the system** - Get a clear picture of how Omnichord works from the top down
- **Navigate the codebase** - Find relevant code quickly with direct file and line references
- **Trace data flow** - Follow the path from keyboard input to audio output
- **Understand threading** - See how the main loop and audio thread interact safely
- **Modify the system** - Make informed changes with knowledge of dependencies

## How to Navigate

### Start Here
1. **[root.md](root.md)** - System-level architecture overview
   - High-level component diagram
   - Module dependency DAG (acyclic view)
   - Threading model
   - Key interfaces
   - Data flow from input to audio

2. **[manifest.md](manifest.md)** - Module index
   - Quick reference table of all modules
   - Status and ownership
   - Links to detailed module docs

### Dive Deeper
After reviewing the system overview, explore individual modules:

- **[modules/main.md](modules/main.md)** - Entry point, game loop, orchestration
- **[modules/input.md](modules/input.md)** - Keyboard handling, chord/strum detection
- **[modules/music.md](modules/music.md)** - Chord theory, frequency calculations, strum plate
- **[modules/audio.md](modules/audio.md)** - Real-time synthesis, voice management, ADSR
- **[modules/display.md](modules/display.md)** - Terminal UI rendering

Each module doc includes:
- Responsibilities and public interfaces
- Internal architecture diagrams
- Dependency diagrams
- Key flows with code references
- Evidence tables mapping components to source code

## Quick Reference

### Architecture Summary
```
Keyboard (evdev) → KeyboardHandler → Chord Detection
                                   ↓
                         StrumPlate → Frequencies
                                   ↓
                         AudioEngine (thread-safe)
                                   ↓
                         VoiceManager → Voice (Oscillator + Envelope)
                                   ↓
                         sounddevice → Audio Output
```

### Key Files
- **Entry point:** `src/omnichord/main.py` - 60 FPS game loop
- **Input:** `src/omnichord/input/keyboard.py` - evdev keyboard handler
- **Music:** `src/omnichord/music/chord.py` - Chord theory and frequencies
- **Audio:** `src/omnichord/audio/engine.py` - Real-time audio synthesis
- **Display:** `src/omnichord/display/ascii_ui.py` - Curses terminal UI

### Threading Model
- **Main thread:** 60 FPS loop polling input, updating logic, rendering UI
- **Audio thread:** sounddevice callback rendering ~11ms audio buffers
- **Thread safety:** `AudioEngine._lock` protects VoiceManager

## Documentation Status

All modules are documented with **complete** status:
- ✅ Main entry and orchestration
- ✅ Input handling (evdev keyboard)
- ✅ Music theory (chords, strum plate)
- ✅ Audio synthesis (engine, voices, envelopes)
- ✅ Display (ASCII UI)

## Maintenance

This SACS documentation was generated on 2026-01-06. To keep it current:

1. **When adding features** - Update relevant module docs with new components
2. **When refactoring** - Update architecture diagrams if data flow changes
3. **When renaming** - Update code references and file paths
4. **When changing threading** - Update root.md threading model section

## Evidence Discipline

All architecture claims in this SACS are grounded in code evidence:
- Every diagram component has corresponding code references
- Every module dependency has import evidence
- Every data flow has file + line number evidence

This is not speculative documentation - it reflects the actual implementation.
