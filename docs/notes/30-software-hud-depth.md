# Software HUD Depth With an Initialised D3D Device

Local investigation, 2026-11-09; based on upstream 9dfa5a5.

## Symptom

With the normal D3D-capable startup and Options > D3D set to OFF, the analog
speedometer loses its dial faces and needles. Digital speed and some static
symbols remain. A completely software startup (-NoD3D) renders the dials.

## Cause And Change

The device Z-buffer flag at 0x53bce4 can remain set while the game's renderer
switch at 0x4dc070 selects software. Model loading and HUD/model drawing then
apply hardware depth adjustments to geometry destined for the software renderer.

Treat the device Z flag as false at the relevant depth-setup reads whenever
D3D is OFF. Seven translated instruction hooks cover model loading/drawing,
the counter, sprite builder and the other HUD builder. The native sprite
builder uses the same condition. The stored device flag is never changed, so
D3D can still be selected normally. Software-only startup keeps its existing path.

A first experiment changed the flag only during the counter draw: it restored
the needles but not dial faces. Covering the model-load depth setup is necessary.

## Validation

On macOS ARM64, isolated Speed Bay sessions used the upstream runtime's scripted
keys and framebuffer snapshots, with separate game settings, saves and logs.
Large immutable game assets were linked to the owned installation.

- Baseline normal startup with D3D OFF: missing analog dial faces/needles reproduced.
- Completely software startup: analog dials visible, including with Alpha enabled.
- Disabling the device Z flag throughout the diagnostic session restored the HUD.
- Final depth-read fix, normal startup with D3D OFF: dial faces and needles visible.
- Final packaged app with D3D ON: race and complete analog HUD verified at 2560x1600.
- Diagnostic D3D-OFF forcing was used only in the probe build and removed from the final source.

The tests enter a race and cycle HUD styles. They establish rendered HUD behavior,
not prolonged gameplay or physical-controller qualification. Audio was muted only
inside the isolated sessions. See the local evidence/software-hud directory for
logs, comparison images, test runner and build output.
