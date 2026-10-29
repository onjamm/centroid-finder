# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mission

Build a Java toolkit that extracts as much verifiable information as possible from an mp4 video or a still image, and reports how reliable each result is. Centroid finding by color is the starting point, not the limit. Anything that can be measured from pixels, motion, or file data is in scope when it passes the guardrails below.

**Information tiers** (each tier is a set of independent analyzers):
- **A. Container and metadata:** codec, frame rate, resolution, timestamps, GPS, provenance credentials (C2PA).
- **B. Classical analysis:** color, edges, blur and noise, background subtraction, optical flow, binarizing and connected groups.
- **C. Learned perception:** detection, segmentation, tracking, depth, pose, using pretrained models.
- **D. Hidden-signal recovery:** pulse from skin color, micro-motion, sound from vibrations in video, shadow-based scene inference. Research-grade, so always report the measured error.
- **E. Provenance and forensics:** signed credentials, edit and compression traces.

Every analyzer returns a result plus its evidence (frame, settings, confidence or error) so a run can be replayed and checked.

## Stack

Java is the core. Maven, Node.js, React and JavaScript are allowed for build tooling, a backend, and a frontend. JavaCV/OpenCV-compatible libraries and pretrained ML models are approved. Before adding any model, dataset or library:
- Check its license and record the source, license and attribution (put it in `data/MANIFEST.md` for datasets).
- Report what was added in the task summary.

## Commands

Commands change as the build changes. Read `README.md` and the build files first. At the time of writing there is no Maven or Gradle; sources are flat in `src/`, and the JUnit runner is `lib/junit-platform-console-standalone-6.0.3.jar`.

```bash
# compile
javac -cp lib/junit-platform-console-standalone-6.0.3.jar src/*.java
# run all tests
java -jar lib/junit-platform-console-standalone-6.0.3.jar -cp src --scan-classpath
# run one test class
java -jar lib/junit-platform-console-standalone-6.0.3.jar -cp src --select-class=<TestClassName>
```

CI requires at least 15 tests to pass on PRs targeting `main`.

## Architecture

Design rule: depend on interfaces, add new behavior as new implementations. Keep the image-to-result flow as a chain of small interfaces, each with its own tests.

Existing modules (a snapshot; `src/` is the source of truth):
- `ImageBinarizer` (impl `DistanceImageBinarizer`, using `ColorDistanceFinder` / `EuclideanColorDistance`): image to black/white grid.
- `BinaryGroupFinder` (impl `DfsBinaryGroupFinder`): grid to sorted list of `Group`.
- `ImageGroupFinder` (impl `BinarizingImageGroupFinder`): composes the two above.
- `ImageSummaryApp`: command-line entry point.

Conventions that existing code relies on:
- Colors are 24-bit `0xRRGGBB` ints. `BufferedImage.getRGB()` is `AARRGGBB`, so strip alpha with `& 0x00FFFFFF`.
- `(row, col)` maps to `(x=col, y=row)`; the origin is top-left and y grows downward.
- `Group` is a record `(int size, Coordinate centroid)`, sorted by size, then x, then y. `Coordinate` is a record `(int x, int y)`.

## Hard Rules

1. **Do not change or edit any existing method or class without explicit permission.** If a change is needed, stop, explain why, and ask. Add new methods or classes instead.
2. **Comment all additions.** Every new method, class, and non-obvious block gets a one- or two-line comment explaining *why*, not just *what*.
3. **Write for two junior developers.** Clear names, small functions, no clever tricks.
4. **Follow existing style.** Match indentation, naming, and file layout already in the repo.
5. **Measure, don't claim.** Every analyzer states how it is measured (for example IoU, error in BPM, SNR) and is tested on synthetic or open data with known answers.

## Privacy and Consent

Pulse, occupancy, face matching and sound recovery are sensitive. Use only footage that is your own, consented, or openly licensed. Process locally, store no biometric data, and make no medical, identity or surveillance claims. Output that is uncertain must say so.

## AI Commit Rules

AI may be used for any file on `video-agentic-approach` and its sub-branches. Every commit that uses AI must start with the message `"AI Used"`. Never mix AI-assisted and human-written code in the same commit.

## Git Rules

- Never work directly on `main`.
- Work only on `video-agentic-approach` and sub-branches of it. Name sub-branches with a hyphen (for example `video-agentic-approach-footage-corpus`), because git cannot create `video-agentic-approach/<name>` while a branch called `video-agentic-approach` exists. Never branch from, switch to, or push to any other branch.
- Pull/rebase from `video-agentic-approach` before starting and before every push.
- Keep changes small and touch as few files as possible.
- Never force-push, rewrite shared history, or discard a teammate's work.
- Do not merge anything yourself; hand the branch to the user to open a PR.
