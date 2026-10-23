# 08 — 3D Animation

## Goal
Make the app feel alive and rewarding for students with one recurring 3D object (the "play-cube": an isometric cube with a play button and page lines). It must never slow the app down.

## Tech choice
- **Rive** (`rive` Flutter package, `.riv` files). Small files, 60 fps, state machines with inputs (tilt, tap, play). Verify the current package version and API on pub.dev before coding.
- Look: 3D-styled artwork (isometric depth, soft shadow) animated in Rive. This gives the 3D feel at low cost on cheap Android phones.
- Fallback for Rive failure or low-end devices: static PNG/WebP of the cube.
- **Not v1:** real-time glTF models (`flutter_scene` is experimental; WebView `model_viewer` is heavy). Revisit after launch if wanted.
- Who makes the `.riv` files: a human designer in the Rive editor, using the Figma file ("3D animation spec" board) as reference. The agent builds the Flutter wrapper and wiring, and must NOT invent `.riv` files.

## Moments
| ID | Where | Behaviour | Duration / plays |
|----|-------|-----------|------------------|
| M1 | Splash | Cube drops in, spins 360° while the progress ribbon fills, fades to next screen | 1.6 s, once per cold start |
| M2 | Home hero card | Idle float (~6 px, 3 s loop); finger tilt follows touch; tap opens resume lesson | loop, pauses off-screen |
| M3 | Payment success | Cube lands with a bounce, confetti (saffron + mist), check appears | 1.2 s, once |
| M4 | Course completed | Cube becomes trophy/certificate, drag to rotate, Share certificate button | once, then interactive |

## Assets
```
assets/animations/
  splash_cube.riv
  home_hero_cube.riv      # state machine "Main", inputs: tiltX, tiltY (number), tap (trigger)
  success_burst.riv       # trigger: play
  complete_trophy.riv     # inputs: tiltX, tiltY, play
assets/images/fallback/   # cube_static.webp, trophy_static.webp
```
Each `.riv` file < 300 KB. Register in `pubspec.yaml`.

## Flutter implementation
- One reusable widget: `core/widgets/hero_3d.dart` → `Hero3D(asset, fallback, stateMachine, onTap)`.
- It loads the `.riv` asynchronously; shows the fallback image until ready or on error.
- It chooses the fallback when: `MediaQuery.disableAnimations` is true, device is low-end (≤ 3 GB RAM or low-performance flag), or the user turned off "Animations" in Profile settings.
- Wrap in `RepaintBoundary`. Pause with `VisibilityDetector` / app lifecycle when not visible.
- Tilt: map `GestureDetector` pan to state-machine inputs; clamp to ±1.
- Animation never blocks interaction: buttons and navigation stay tappable.
- Play-once moments (M1, M3) store a flag so they do not replay on rebuild.

## Performance budget
- 60 fps on a 3 GB RAM Android 8+ device; no dropped frames on Home scroll.
- Cold start must stay < 3 s: Splash animation overlaps loading, it does not add to it.
- APK size impact < 1.5 MB total for all animations.

## Acceptance criteria
- Each moment M1–M4 matches the table above.
- Reduce-motion and low-end fallback show the static image, same layout, no jump.
- App works fully if `.riv` fails to load (test by renaming the asset).
- Profile has an "Animations" on/off switch (default on).
