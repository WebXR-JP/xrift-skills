---
name: xrift-world-editing
description: Guide for editing XRift worlds from the browser through WebMCP tools. Covers the nine tools exposed on app.xrift.net (place, update, remove, group, undo, and four read tools), the coordinate and rotation conventions, placement anchors, limits, and the rules that keep the agent from disrupting the human user.
---

# XRift World Editing via WebMCP

A guide for AI agents that edit XRift worlds through the [WebMCP](https://developer.chrome.com/docs/ai/webmcp)
tools the page registers while the user is inside an instance they can edit.

## References

- [Tool Reference](references/tool-reference.md) - Full input/output schema for all nine tools, placeable types, and error messages

## Critical Rules

1. **Call `get-viewer-context` before placing anything relative to the user.** It returns
   `viewDirection` (a unit vector), `crosshairSurface` and `playerPosition`. Never guess where the
   user is or what they are looking at.
2. **`crosshairSurface` can be `null`; `playerPosition` effectively cannot.** Looking at the sky or
   at nothing within range leaves no surface, and `anchor: 'crosshair'` then fails. Fall back to
   `anchor: 'player'`, which is the user's **feet**, so `y: 0` sits on the ground they stand on.
3. **Never convert direction into an angle yourself.** To place something N meters in front of
   the user, add `viewDirection * N` to the position. To make an object face the user, use
   `rotationDegreesToFaceViewer` as `rotationDegrees.y` verbatim. These differ by 180° — mixing
   them up puts objects behind the user, facing away.
4. **Rotation is in degrees, position is in meters.** One unit is one meter. Do not send radians.
5. **Call `list-placeable-types` before the first `place-objects`.** Type names and the valid
   range of each dimension come from there, not from memory.
6. **One call, many objects.** `place-objects` takes up to 50 objects. Placing a 15-part bench
   in one call is one undo step for the user; 15 calls are 15 steps.
7. **If a call fails, nothing was placed.** Validation is all-or-nothing. Fix the arguments from
   the error message and call again — do not assume a partial result and try to patch it up.
8. **Never fight the user for control.** Tools refuse while the user is placing or moving an
   object, or answering a confirmation dialog. When refused, say so and wait; do not retry in a
   loop.
9. **Re-read before editing.** `update-objects` and `remove-objects` take ids from `get-scene`.
   Other people are editing the same world concurrently, so ids go stale.
10. **Images come from `list-world-images`, never from a URL.** Pass the image's `id` as
    `imageAssetId`. There is no way to show an image from another site; if the one the user
    wants is not listed, ask them to upload it from Asset Management in edit mode.
11. **A world has at most one spawn point.** If `get-scene` with `type: 'spawn-point'` returns
    one, move it with `update-objects` instead of placing another. Put it (and sit areas) on
    solid ground — use `anchor: 'crosshair'` or `'player'` so it sits on a real surface.

## Where the tools exist

Registered only while **all** of these hold:

- The page is `https://app.xrift.net` (Chrome 149+; the Origin Trial runs until 2026-11-17)
- The user is inside an instance
- The world is browser-editable (EDITOR type — not a code-built CODE world)
- The user owns the world or was granted edit access

Edit mode is **not** required. The tools are there while the user walks around.

If the tools are missing, the reason is one of the above. Tell the user which to check rather
than retrying.

## Coordinates

Right-handed, Y up, one unit = one meter.

| Concept | Where it comes from |
|---|---|
| Absolute position | `anchor: 'world'` — position is from the world origin |
| Relative to the user's gaze | `anchor: 'crosshair'` — position is from the surface under the crosshair |
| Relative to where the user stands | `anchor: 'player'` — position is from their feet |
| "N meters in front of the user" | `anchor: 'player'` with `position = viewDirection * N` |
| "Facing the user" | `rotationDegrees.y = rotationDegreesToFaceViewer` |

`get-scene` always reports **world** coordinates, even for objects inside a group. When you feed
a position from `get-scene` back into `update-objects`, it is interpreted the same way — no
conversion needed.

## Limits

| | |
|---|---|
| Objects per call | 50 |
| Objects per world | 500 (`remainingCapacity` in `get-viewer-context`) |
| Coordinate range | ±500 m per axis |
| Scale | 0.1 – 5 |
| Dimensions | Per type; see `list-placeable-types` |

## A typical session

```
1. get-viewer-context      → where the user is looking, how much room is left
2. list-placeable-types    → type names and dimension ranges
3. place-objects           → build it in one call
4. group-objects           → combine the parts into one object (optional)
```

To modify something that already exists:

```
1. get-scene (filter with type / ids / limit)  → current ids and positions
2. update-objects / remove-objects             → act on those ids
```

## Not yours to touch

- **The user's selection.** Tools never change it, and there is no tool to set it. If the user
  needs to see what you placed, tell them to press 選択 (Select) on the banner that appeared.
- **Edit mode.** Never ask the user to enter it; the tools do not need it.
- **Objects someone else is editing.** They are silently excluded and reported back to you
  (`lockedByOther: true` in `get-scene`). Do not retry them — wait for the other person.
- **The user's own undo history.** `undo-last-agent-edit` only reverts your own last edit. If the
  most recent change was made by the user, it refuses; tell the user to press `Cmd/Ctrl+Z`.

## Reporting back

The user sees a banner, not your tool output, so summarize what you did in plain language:
what you built, roughly where, and how many objects. Pass along any counts the tool returned
about objects that were skipped — "3 objects someone else is editing were left alone" is
actionable; silence is not.

Keep the ids of what you placed in context. The user's next sentence is usually "move it a bit
to the left".
