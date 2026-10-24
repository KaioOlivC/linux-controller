# Linux Controller

A gamepad-driven interface for controlling a Linux desktop using an **8BitDo Pro 3 controller**. The goal is to operate the mouse, type text, and manage windows in i3 without needing a conventional keyboard or mouse.

The project is inspired by ergonomic computing, alternative keyboard layouts, and rethinking how we interact with computers.

## Goals

- Control the mouse, keyboard, and i3 window manager with one controller.
- Make text entry practical without requiring simultaneous face-button chords.
- Keep controls consistent, predictable, and comfortable to use.
- Make mouse movement and keyboard input responsive without accidental repeats.
- Keep the implementation modular and easy to experiment with.
- Develop incrementally, testing each feature before adding more complexity.

## Target Environment

- **Operating system:** Arch Linux
- **Window system:** X11
- **Window manager:** i3
- **Controller:** 8BitDo Pro 3
- **Connection:** Controller communicates wirelessly with its 2.4 GHz receiver, which is connected to the computer through a USB dock.
- **Implementation language:** Python (proposed)

The exact controller input interface, button indices, and input-injection backend still need to be verified on the target machine.

## Interaction Model

The D-pad selects one of four operating modes. Each mode assigns its own actions to the controller's buttons and sticks.

The planned modes are:

1. Mouse
2. Keyboard
3. i3 window management
4. A fourth mode, to be decided

Mode changes must be handled safely. Switching modes should not leave mouse buttons or keyboard keys logically held down.

## Mouse Mode

| Input | Action |
|---|---|
| Left stick | Move cursor |
| Right stick | Control mouse scrolling |
| A | Left click |
| B | Right click |
| X | Mouse back button |
| Y | Mouse forward button |
| R3 | Middle click |

### Mouse behavior

- Cursor movement should support adjustable sensitivity and acceleration.
- Stick dead zones should prevent unwanted movement caused by analog drift.
- Scroll sensitivity should be independently adjustable.
- Holding A while moving the cursor should support click-and-drag.
- Horizontal scrolling is desirable if practical.

## Keyboard Mode

The keyboard mode provides text entry and keyboard shortcuts through sequential button selection and modifier buttons.

### Modifier and special-button assignments

| Input | Action |
|---|---|
| Left stick | Arrow keys |
| Start | Enter |
| Select | Backspace |
| L1 | Shift modifier |
| L2 | Ctrl modifier |
| R1 | Space when tapped; modifier when held |
| R2 | Tab modifier |
| Face buttons (A, B, X, Y) | Sequential character selection |

Alt and Super are not required in the initial keyboard implementation. Super is reserved primarily for i3-related operations in the overall design.

### Modifier behavior

- L1, L2, R1, and R2 use hold-to-modify behavior.
- R1 inserts a space when released without being used as a modifier.
- If R1 participates in another action, no space should be inserted.
- R1 combined with a face button may have its own explicitly defined action.
- Modifier combinations must not generate unintended standalone key presses.
- Generated key presses must always be released correctly, including when changing modes.

### Character selection

Typing will use sequential face-button presses, **not simultaneous face-button chords**.

The proposed design is a two-stage selection system:

1. Press a face button to select a character group.
2. Press a second face button to select a character within that group.

Four face buttons provide up to 16 combinations per layer.

Character assignments remain to be designed. Common letters and frequently used characters should receive convenient, memorable mappings. Additional layers can accommodate uppercase letters, digits, punctuation, and symbols.

Space, Backspace, Enter, and Escape should remain easy to access.

### Key-repeat policy

For the initial implementation:

- Holding a button must not repeatedly generate characters or actions.
- Each recognized press generates at most one logical action.
- A new action requires a release followed by a new press.
- Keyboard events must use correctly paired key-down and key-up events.

Key repeat may be reconsidered later as an optional feature.

## i3 Window-Management Mode

| Input | Action |
|---|---|
| Left stick | Navigate between windows |
| Right stick | Move or reposition the focused window |
| A | Horizontal split layout |
| B | Vertical split layout |
| X | Close focused window |
| Y | Toggle fullscreen |
| L1 | Previous workspace |
| R1 | Next workspace |
| L2 | Toggle floating mode |
| R2 | Toggle floating/grid behavior |

### i3 behavior

- Window navigation should use directional input with controlled repeat timing.
- Window movement must distinguish between moving a window within the tiling layout and repositioning a floating window.
- X should request a normal window close rather than forcibly terminating the application.
- L2 should use i3's native floating toggle.
- R2's floating/grid behavior needs further design. A possible implementation is arranging floating windows into a custom grid.
- Optional notifications or status indicators may communicate the current mode and relevant window state.

## Architecture

The project will be implemented as **one Python application with separate modules**.

The proposed architecture separates controller input, mode handling, and desktop actions:

```text
linux-controller/
├── README.md
├── .gitignore
├── pyproject.toml
├── src/
│   └── linux_controller/
│       ├── __init__.py
│       ├── main.py
│       ├── controller.py
│       ├── modes.py
│       ├── keyboard.py
│       ├── mouse.py
│       └── i3.py
└── tests/
```

This is an initial proposed structure, not a requirement to create every file immediately.

### Module responsibilities

- `main.py`: application entry point and lifecycle.
- `controller.py`: reads controller inputs and tracks button/stick state.
- `modes.py`: manages mode switching and dispatches actions.
- `keyboard.py`: character selection, modifiers, and keyboard events.
- `mouse.py`: cursor movement, scrolling, clicks, and dragging.
- `i3.py`: window-management commands and workspace navigation.
- `tests/`: automated tests for input interpretation, modifier logic, and mode transitions.

The input layer should remain separate from the action backends so that mappings and implementations can evolve independently.

## Development Plan

Development will proceed incrementally:

1. Initialize the Python project and establish the module structure.
2. Detect the controller and verify its reported buttons, axes, and connection behavior.
3. Implement mode management and log interpreted inputs.
4. Implement and test keyboard modifiers and sequential character selection.
5. Add keyboard event injection and verify key-release safety.
6. Implement mouse movement, scrolling, clicking, and dragging.
7. Implement i3 window-management actions.
8. Add configuration options, tests, and usability refinements.

The initial prototype should **log interpreted actions before injecting keyboard or mouse events**. This allows the input logic to be validated safely before it controls the desktop.

## Design Principles

- Prefer explicit, predictable behavior over clever but ambiguous mappings.
- Avoid simultaneous face-button chords.
- Prevent accidental key repeats and stuck keys.
- Make sensitivity, dead zones, and character mappings configurable where practical.
- Keep the application modular without introducing unnecessary complexity.
- Test each feature on the actual target system.
- Preserve a clean Git history and commit working milestones.

## Open Questions

- Which Linux interface and library should read the controller?
- Which input-injection backend is most appropriate for Arch Linux on X11?
- What are the controller's actual button and axis indices?
- What is the final character map for the two-stage selection system?
- How should shifted characters and additional symbol layers work?
- What exactly should the R2 floating/grid action do?
- What should the fourth mode be?
- Should mode changes trigger a visual indicator or notification?
- Which settings should be user-configurable?

These decisions should be updated as the implementation evolves.

## Project Status

**Current stage:** Initial design and repository setup.

The interaction model and major button assignments have been agreed upon. Implementation has not yet begun.

