# Lab 2: Brick Breaker

Your second complete game, built solo: a paddle at the bottom, a marionberry
that ricochets off real physics, and a wall of bricks that shatters one signal
at a time. Then at least one twist that makes it yours.

Lab 1 did its own math. Lab 2 hires the physics engine. Everything from the
collision lecture goes to work here: custom input actions, `CharacterBody2D`
movement on the physics clock, `StaticBody2D` obstacles, and collision layers
and masks. The brick grid also puts this week's material on the field:
instancing scenes from code, and a signal the whole game listens to.

## Getting started

1. Accept the assignment link (posted on Canvas), then clone **your** repo:

   ```bash
   git clone <your-repo-url>
   ```

2. Open `project.godot` in Godot (standard build). Run the project. An empty
   dark window with three violet walls is correct: that is your arena, missing
   its floor on purpose.
3. Open **`GUIDE.md`** (in this repo) and build milestone by milestone.

## What is already here

- `walls.tscn`, a prebuilt `StaticBody2D` with the top and side walls (you
  built one of these in class; this one you get for free). There is no bottom
  wall. The floor is where marionberries go to be missed.
- `assets/paddle.png` (gold, horizontal), `assets/ball.png` (the marionberry),
  and `assets/brick.png` (near-white on purpose: you will tint it with
  `modulate`)
- `main.tscn`, an empty `Main` scene, already set as the main scene
- This README and `GUIDE.md`

Everything else you build yourself.

## Requirements

Your submitted game must have, at minimum:

1. A **paddle** the player controls through **custom input actions** (not the
   `ui_*` built-ins), stopped by the side walls.
2. A **ball** moved by the physics engine that **bounces** off walls, paddle,
   and bricks, and **serves again** after a miss.
3. A **grid of bricks** instanced from code. Bricks disappear when hit and
   report their demise through a **signal**; a visible **score** counts it
   (the Output panel is fine, an on-screen Label is a stretch goal).
4. **Named collision layers**, with each scene's layer and mask set on
   purpose. Be ready to explain who notices whom.
5. At least **one custom feature** you chose and built (see the "Make it
   yours" menu at the end of `GUIDE.md`). Name it in **Your Submission**.
6. Class code style: typed variables, `@export` for tunables, and motion on
   the correct clock.

The guide shows one way to build the core. Deviate freely as long as the
requirements are met.

## How to submit

**Pushing to `main` is the submission.** Commit and push at every milestone:

```bash
git add -A
git commit -m "M2: the berry ricochets"
git push
```

---

## Your Submission

*Fill this section in before the deadline. It is part of the grade.*

**Screenshot of your game:**

> [Replace this line with a screenshot. Commit an image to the repo and embed
> it: `![screenshot](shot.png)`]

**My custom feature(s):**

> [What did you add or change to make it yours? A sentence or two each.]

**My layer map:**

> [In one or two sentences: which layers exist in your game, and who masks
> whom? Explaining this is part of the lab.]

**One thing that surprised me:**

> [A bug, a behavior, a Godot thing. What did you not expect?]

