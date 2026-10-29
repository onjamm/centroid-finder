# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Java image-analysis tool that binarizes an image by color proximity and finds connected pixel groups (centroids). The goal is to evolve this into a video/timelapse object-tracking system. All tracking logic stays in Java; Node.js + React will be added later for a frontend.

**Allowed stack:** Java, Maven, Node.js, React, JavaScript only. Ask before adding any ML model or library.

## Commands

There is no Maven or Gradle. All source files live flat in `src/`. The JUnit standalone runner is at `lib/junit-platform-console-standalone-6.0.3.jar`.

**Compile:**
```bash
javac -cp lib/junit-platform-console-standalone-6.0.3.jar src/*.java
```

**Run all tests:**
```bash
java -jar lib/junit-platform-console-standalone-6.0.3.jar -cp src --scan-classpath
```

**Run a single test class:**
```bash
java -jar lib/junit-platform-console-standalone-6.0.3.jar -cp src --select-class=DfsBinaryGroupFinderTest
```

**Run the app end-to-end (produces `binarized.png` and `groups.csv`):**
```bash
javac -cp lib/junit-platform-console-standalone-6.0.3.jar src/*.java && java -cp src ImageSummaryApp sampleInput/squares.jpg FFA200 164
```

CI requires at least 15 tests to pass on PRs targeting `main`.

## Architecture

The pipeline is a straight chain of interfaces and their single implementations:

```
BufferedImage
  └─ ImageBinarizer.toBinaryArray()       → int[][] (0=black, 1=white)
       impl: DistanceImageBinarizer       uses ColorDistanceFinder + target color + threshold
               └─ ColorDistanceFinder.distance()
                    impl: EuclideanColorDistance
  └─ ImageBinarizer.toBufferedImage()     → BufferedImage (black=0x000000, white=0xFFFFFF)

int[][]
  └─ BinaryGroupFinder.findConnectedGroups()  → List<Group> (sorted descending)
       impl: DfsBinaryGroupFinder             iterative DFS / BFS via ArrayDeque

BufferedImage
  └─ ImageGroupFinder.findConnectedGroups()   → List<Group>
       impl: BinarizingImageGroupFinder       composes ImageBinarizer + BinaryGroupFinder

ImageSummaryApp (main)  wires everything together, writes binarized.png + groups.csv
```

**Data types:**
- Colors are 24-bit `0xRRGGBB` ints (no alpha). `BufferedImage.getRGB()` returns 32-bit `AARRGGBB`; strip alpha with `& 0x00FFFFFF` before passing to `ColorDistanceFinder`.
- Coordinate system: `(row, col)` maps to `(x=col, y=row)`. Top-left is `(x=0, y=0)`; y increases downward.
- `Group` is a record `(int size, Coordinate centroid)`. Sorted descending by size, then x, then y.
- `Coordinate` is a record `(int x, int y)`.

## Hard Rules

1. **Do not change or edit any existing method or class without explicit permission.** If a change is needed, stop, explain why, and ask. Add new methods or classes instead.
2. **Comment all additions.** Every new method, class, and non-obvious block gets a one- or two-line comment explaining *why*, not just *what*.
3. **Write for two junior developers.** Clear names, small functions, no clever tricks.
4. **Follow existing style.** Match indentation, naming, and file layout already in the repo.

## AI Commit Rules

AI may only be used to help write **test files**. Every commit that uses AI must:
- Start with the message `"AI Used"`
- Only create or alter test files

Never mix AI-assisted and human-written code in the same commit.

## Git Rules

- Never work directly on `main`. Use a short-lived feature branch per task.
- Pull/rebase from `main` before starting and before every push.
- Keep changes small and touch as few files as possible.
- Never force-push, rewrite shared history, or discard a teammate's work.
- Do not merge anything yourself; hand the branch to the user to open a PR.

## Branch Restriction

- Stay on the `video-agentic-approach` branch. Do not switch to, commit to, or push to any other branch (including `main`).
- Create any new work branches only as sub-branches off `video-agentic-approach` (e.g. `video-agentic-approach/<task-name>`), never off any other branch.
