# The Build Guide

One way to build the brick breaker, milestone by milestone. Every milestone
ends with something you can run. Commit and push at each one.

Ground rules, same as Lab 1:

- **This is a guide, not a script.** Boxes marked *Dials* hold values meant to
  be tuned; boxes marked *Your call* mark places where the design is up to
  you. Deviate freely as long as the README requirements are met.
- **Ask "which class moment is this?"** The paddle is Blitz's movement turned
  sideways. The ball is the collision lecture. The brick grid is the berry
  spawner from week one, grown up. The smashed signal is the doorbell you
  already know.

The plan: a paddle at the bottom, real physics everywhere, no floor. Miss the
berry and it is gone.

---

## Milestone 0: Open the arena

1. Clone your repo and open `project.godot` in Godot.
2. Run the project. Three violet walls and no floor is correct. Stop it.

---

## Milestone 1: A paddle with real collision

**Actions and layers first.** Two trips into Project Settings:

1. **Project → Project Settings → Input Map** tab. Add an action named
   **`move_left`**, then use its **+** button to bind the **Left Arrow**, and
   again to bind **A**. Add **`move_right`** with **Right Arrow** and **D**.
   Close the dialog.
2. **Project → Project Settings → General**, search **"layer names"**, open
   **Layer Names → 2D Physics**. Name layer 1 **`world`**, layer 2
   **`paddle`**, layer 3 **`bricks`**, layer 4 **`ball`**.
3. Both live in `project.godot`. Commit that file; your input scheme and
   layer map are part of the game.

**The scene.**

1. **Scene → New Scene**, root: **Other Node → `CharacterBody2D`**, renamed
   **`Paddle`**. It moves and things bounce off it, so it is a body.
2. Add a **`Sprite2D`** child and drag **`assets/paddle.png`** onto its
   **Texture** slot.
3. Select `Paddle` again and add a **`CollisionShape2D`** child. In the
   Inspector choose **Shape → New RectangleShape2D** and size it to cover the
   sprite (about 96 x 16).
4. Still on the `Paddle` root, find **Collision** in the Inspector. Set
   **Layer** to `paddle` only (box 2) and **Mask** to `world` only (box 1).
   In English: "I am a paddle; walls stop me."
5. Save as **`paddle.tscn`**.

**The script.** Right-click `Paddle`, choose **Attach Script**, keep
`res://paddle.gd`, and make it:

```gdscript
extends CharacterBody2D

@export var speed: float = 520.0

func _physics_process(delta: float) -> void:
    var direction := Input.get_axis("move_left", "move_right")
    velocity = Vector2(direction * speed, 0.0)
    move_and_slide()
```

Notice what is missing: the `clamp` line from Lab 1. The side walls stop the
paddle now, because `move_and_slide` refuses to push a body through the
`world` layer. Less code, more correct.

**Put it in the world.** Open **`main.tscn`**, select the `Main` root, and
instance two scenes into it (the chain-link icon): **`walls.tscn`**, and
**`paddle.tscn`** at position `(576, 600)`. Save and run. The paddle slides
and the walls refuse to let it leave.

> **Dials:** `speed`. A slow paddle makes a hard game.
>
> **Your call:** paddle on a side wall instead, Pong-style vertical? Rotate
> the whole design; the guide adapts the same way Lab 1 did.

**Commit:** `git add -A`, commit, push.

---

## Milestone 2: A berry that ricochets

**The scene.** New scene, root **`CharacterBody2D`** named **`Ball`**, with a
`Sprite2D` (**`assets/ball.png`**) and a `CollisionShape2D` (**New
CircleShape2D**, radius about 12). On the `Ball` root set **Collision →
Layer** to `ball` (box 4) and **Mask** to `world`, `paddle`, and `bricks`
(boxes 1, 2, 3). In English: "I notice everything solid. Nothing needs to
notice me." Save as **`ball.tscn`**.

**The script** (`res://ball.gd`):

```gdscript
extends CharacterBody2D

@export var speed: float = 420.0

func _ready() -> void:
    serve()

func serve() -> void:
    position = Vector2(576, 480)
    var direction := Vector2(randf_range(-0.6, 0.6), -1.0).normalized()
    velocity = direction * speed

func _physics_process(delta: float) -> void:
    var collision := move_and_collide(velocity * delta)
    if collision:
        velocity = velocity.bounce(collision.get_normal())

    if position.y > get_viewport_rect().size.y + 24.0:
        print("Miss!")
        serve()
```

Three things worth reading twice:

- **`move_and_collide` is `move_and_slide`'s stricter sibling.** Slide cancels
  the blocked part of your motion and keeps going (right for walkers). Collide
  **stops at the first contact and hands you a report** (right for bouncers).
- **The `* delta` is back.** `move_and_slide` applies delta for you;
  `move_and_collide` takes a raw motion vector, so you supply it. Same units
  lesson, third week in a row.
- **`bounce(normal)` does the reflection math.** The collision report knows
  which way the surface faced; `bounce` mirrors your velocity across it. That
  one line is the whole ricochet.

**Instance `ball.tscn` into `main.tscn`** and run. The berry ricochets off
walls and paddle, and falls past the missing floor to a printed "Miss!" and a
fresh serve.

> **Dials:** `speed`, the serve angles, the serve position.
>
> **Your call:** serve on a key press instead of instantly
> (`Input.is_action_just_pressed`), or serve toward a random corner.

**Commit.**

---

## Milestone 3: One brick

**The scene.** New scene, root **`StaticBody2D`** named **`Brick`** (it never
moves; it just *is*, until it isn't). Add a `Sprite2D` with
**`assets/brick.png`** and a `CollisionShape2D` (**New RectangleShape2D**,
about 64 x 24). On the root set **Collision → Layer** to `bricks` (box 3) and
clear the **Mask** entirely. Bricks notice nobody; the ball notices them.
Save as **`brick.tscn`**.

**The script** (`res://brick.gd`):

```gdscript
extends StaticBody2D

signal smashed

func take_hit() -> void:
    smashed.emit()
    queue_free()
```

**Teach the ball to smash.** Back in **`ball.gd`**, grow the collision branch
of `_physics_process` (add the two new lines inside the `if collision:` block):

```gdscript
    var collision := move_and_collide(velocity * delta)
    if collision:
        velocity = velocity.bounce(collision.get_normal())
        var collider := collision.get_collider()
        if collider.has_method("take_hit"):
            collider.take_hit()
```

`has_method` is a quiet piece of design: the ball does not ask "are you a
brick?", it asks "can you be hit?". Walls and the paddle cannot, so they only
bounce. Anything you invent later that answers `take_hit` becomes smashable
with zero changes to the ball.

**Test it:** instance a single `brick.tscn` into `main.tscn` somewhere in the
upper half, run, and break it.

**Commit.**

---

## Milestone 4: The wall of bricks

Nobody places 48 bricks by hand. Delete your hand-placed test brick, then
right-click the `Main` root, choose **Attach Script** (`res://main.gd`), and
make it:

```gdscript
extends Node2D

const BRICK := preload("res://brick.tscn")
const ROW_COLORS: Array[Color] = [
    Color("2de2e6"),
    Color("45f0a1"),
    Color("ffd23f"),
    Color("ff2e97"),
]

var score: int = 0

func _ready() -> void:
    for row in range(4):
        for col in range(12):
            var brick := BRICK.instantiate()
            brick.position = Vector2(120 + col * 80, 90 + row * 40)
            brick.modulate = ROW_COLORS[row]
            brick.smashed.connect(_on_brick_smashed)
            add_child(brick)

func _on_brick_smashed() -> void:
    score += 10
    print("Score: ", score)
```

- Nested loops stamp out the grid: the berry spawner from week one, grown a
  second dimension.
- **`modulate` tints a node's drawing.** One near-white sprite plus four
  colors equals four kinds of brick and zero extra art.
- `brick.smashed.connect(...)` is the same subscription you wrote for the
  marionberries. Forty-eight bricks announce; one function listens; none of
  them know the score exists.

Run it. Break the wall. Watch the score climb in the Output panel.

> **Dials:** rows, columns, spacing, points per brick.
>
> **Your call:** pyramid layout, checkerboard gaps, or rows worth different
> points (`score += (4 - row) * 10` makes high bricks precious).

**Commit.**

---

## Milestone 5: Make it yours (required)

Pick **at least one**, or invent your own of similar size. Name it in the
README's **Your Submission** section.

- **Lives and game over.** Three misses ends the game (a print is fine; a
  restart is better).
- **Tough bricks.** Give bricks hit points: `take_hit` decrements, darkens
  the tint (`modulate = modulate.darkened(0.3)`), and only smashes at zero.
- **Angle control.** After bouncing off the paddle, bend `velocity.x` by how
  far from the paddle's center the ball struck. This is the shot-making skill
  of every good brick breaker.
- **Power-up drop.** A smashed brick sometimes drops a falling `Area2D` the
  paddle can catch (a wider paddle, a faster ball, an extra life). The berry
  from Lab 1 knows how.
- **Speed ramp.** Every smash nudges the ball faster; cap it with
  `limit_length`.
- **On-screen score.** A `Label` updated from `main.gd`. Thursday's lecture
  makes this feel obvious.
- **Win state.** Count the bricks; when none remain, declare it. "CLEAR!" has
  been enough since 1976.
- **Reskin it.** New sprites, new palette, new fiction. The mechanics do not
  care.

---

## Nothing happened?

| Symptom | Cause and cure |
| --- | --- |
| Ball sails through bricks | The ball's **Mask** is missing box 3, or the brick's **Layer** is not 3. Whose layer, whose mask? |
| Ball never bounces at all | You used `move_and_slide` in the ball, or forgot `* delta` in `move_and_collide` (the motion is huge and tunnels). |
| Ball jitters inside a wall | It spawned overlapping something. Serve from open space, and bounce with `collision.get_normal()` rather than flipping a sign yourself. |
| Paddle escapes the arena | Its **Mask** is missing `world`, or the walls scene was never instanced into `Main`. |
| Brick ignores the hit | `take_hit` is misspelled, or the smash lines sit outside the `if collision:` block. Print `collision.get_collider()` and look. |
| "Action already exists" | You are creating actions on a branch that already has them. Check which branch you are on. |
| Editor shows stale actions or layers after a checkout | Project Settings do not reload from disk. Run **Project → Reload Current Project** without saving first. |

The checklist, week 3 edition: which scene ran, is the node in it, is the
script saved, whose layer, whose mask?
