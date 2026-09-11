# exterior-design-mocks — residential

Exterior-to-interior UX prototypes for CoStar, Homes.com side. These explore how
someone moves from an aerial capture of a site into an interior tour of one home
inside it.

They are design artifacts, not production code. Read them as arguments about
interaction, and change them the way you would change a drawing.

**This repo is a scoped-down copy.** The original carries LoopNet commercial
prototypes alongside these (`commercial/a/`, `commercial/b/`). They are not here
on purpose. Do not recreate them, do not link them from the landing page, and
treat any reference to them you find in a comment as a leftover worth deleting
rather than a missing folder worth restoring.

## Ground rules

- **Single-file HTML, no build step.** Each prototype is one `index.html` with
  inline CSS and one inline `<script>`. Open it in a browser; that is the whole
  toolchain. Do not add bundlers, package files, or split into modules.
- **Three.js r128 from cdnjs**, loaded as a plain `<script src>`. `THREE` is a
  global. r128 predates a lot of the modern API — notably `CapsuleGeometry`
  (r142) does not exist. Check before reaching for anything recent.
- **Comments explain the reasoning, not the mechanics.** The existing comments
  say why a number is what it is and what went wrong before it got there. Match
  that. A comment that restates the line below it is noise.
- **No emoji, no decorative headers in code.**

## Layout

```
index.html              landing page — one card, one prototype
homes/a/                multi-entry:  residential community, floors per home
PROTOTYPE-RATIONALE.md  STALE. Predates several rounds of changes. Verify before trusting.
```

**`homes/b/` and `homes/a-loading/` were removed from the working tree** partway
through a session, outside of Claude. Nothing here recreates them, and the
landing page's single-entry card was removed to match. If `homes/b/` comes back,
the card has to be written again — do not scaffold a new prototype from scratch
to fill the slot.

What they were, so the history in these notes still parses: `homes/b/` was the
single-entry strategy — one capture, one tourable space, one pin, no floor
stack, the argument for it being lower engineering effort. `homes/a-loading/`
was a full copy of `homes/a/` used to iterate on the load sequence, kept whole
because `startReveal()` hands off to a real scene and a stripped harness has
nothing to hand off to.

There used to be another, `homes/a-hifi/`: a fidelity study carrying image-based
lighting off the procedural sky, procedural normal maps on siding and roofing,
and per-material `envMapIntensity`. It won, so it was promoted — `homes/a/` *is*
that build now, and the study folder is gone. If you are looking for the
reasoning behind the environment map, it is in the image-based lighting comment
in `homes/a/`, not in a folder that no longer exists.

### The A / B distinction

**A (multi-entry)** — the capture contains several separately tourable homes.
Pins marked assets inside a parcel; picking one opened a floor list, and picking
a floor sectioned the house and opened that level. This is the only strategy
still in the tree; B is described above, under Layout.

**The pins are gone, and with them the way in.** See below — A's argument is now
made by the capture and the orbit alone.

## Architecture inside a prototype

Two scenes in one renderer, cross-faded rather than cut:

- `exterior` — the aerial capture. Orbit rig, POI pins, the section-cut system.
- `interior` — the tour. Walk / dollhouse / floorplan views.

`enterInterior()` fades out, swaps scenes under cover, and fades in. `savedAerial`
holds the aerial rig state so leaving the tour returns you to where you stood.

### Camera rig

One `rig` object holds both current and **goal** state (`gTheta`, `gPhi`,
`gTarget`, `gRadius`). Every camera change writes the goal; the animation loop
eases toward it with frame-rate-independent damping:

```js
const k = 1 - Math.pow(0.0016, dt);
```

So `flyTo(focus)` never moves the camera itself — it sets a destination. Write
new camera behaviour as a focus function returning `{target, theta, phi, radius}`.

`phi` is measured **off vertical**. Larger = closer to level. `1.30` is a
near-level elevation; `0.86` (~40° above the horizon) is the plan-reading tilt
used for a sectioned floor.

`nearestAngle(cur, goal)` takes the short way round so headings never spin.

### Aerial controls

Bottom-left: a pill holding **play/pause**, and a **reset** button beside it.
Play/pause is the auto-orbit. Reset returns the framing — `HOME` target, theta,
phi, radius — without restarting the motion.

`rig.auto` is the orbit. The button reports it rather than owning it, because
plenty of things stop the orbit without going through the button: a drag or a
WASD pan (`nudge()`), opening a pin (`selectPOI` → `flyTo`), entering a tour.
`syncPlay()` reads `rig.auto`, and the frame loop watches the flag against
`lastAuto` and repaints on any change — which is what saves every one of those
call sites from having to remember the button exists.

`rig.userPaused` is narrower: it marks a stop the user *asked* for — pausing, or
resetting. `closeCard({resume:true})` hands the orbit back only when it is false,
so dismissing a card cannot undo a deliberate pause.

There is no guided tour and no prev/next transport. Stepping between pins lives
on the card's own arrows and the `[` / `]` keys. There is no orbit toggle
separate from play/pause; there used to be, and both are gone in favour of the
one control.

Bottom-right (`#aerialRight`) is Share, overflow, Ask AI, in that order. Ask AI
is decorative — it swallows its own events so a click cannot reach the orbit rig
behind it. Share opens `#shareModal`. Share's button carries its own
`0 0 38 38` viewBox and a stroked `.gly` glyph, because the sprite the other two
crop has no share artwork and `.btn-fg` fills rather than strokes.

`#shareModal` is the one true modal here — `z-index:70`, above the veil and both
panels, with a scrim that takes the scene. That is the opposite of `#spaceInfo`
on purpose: sharing is a task you finish and leave, not a reference you keep open
beside the capture. Escape unwinds outermost first — share, then space info, then
the scene — each guard returning so one press never takes two things.

The live part is the **Link to this location** checkbox: ticking it writes
`rig.gTheta/gPhi/gRadius` into the URL, so the field changes as you orbit. That
is the whole argument the modal exists to make — share the space, or share the
shot — and the reason the link is fabricated rather than omitted. Copy tries the
async clipboard, then falls back to selecting the field and `execCommand`,
because a `file://` page is refused the first path outright and that is how this
prototype is usually opened.

**The five destination circles are placeholders**, lettered in the page's own
font. They are deliberately not the services' brand marks; the names live in
`aria-label`. Drop in the official SVG from each brand kit before this is shown
as anything but a mock.

### What the aerial view no longer has

Two systems were removed outright, and their absence is deliberate — do not add
them back as "missing polish":

- **The compass tape** (`#compass` / `#compassTape`, `buildCompass`,
  `updateCompass`). A heading strip along the top that scrubbed as the view
  turned. The bearing was `-rig.theta`, taking `-Z` as north.
- **The first-run captions** (`#hint` / `#hintMore` and the whole `setHint`
  phase machine). Three phases: pointer gestures, then WASD/QE keys, then an
  arrow pointing at the overflow menu. Phase two was earned by a real drag
  rather than timed, with a 7s fallback.
- **The settings gear** (`#settingsBtn`), top-right, opposite the brand lockup.
  Decorative, and the only chrome that persisted across both modes. Share took
  over as the tool in that corner of the job, in the bottom-right cluster.
- **The POI pins and their leader line** (`#poiLayer`, `#leaderLayer`,
  `updatePins`, `drawLeader`, `placeCardByPin`, `stepPOI`). Kite markers
  projected onto the capture each frame and occlusion-tested against the
  buildings; the card either docked in the corner or chased its pin with a line
  running back to the dot.

### Nothing opens the interior any more

Removing the pins removed the only way in, deliberately. The aerial view is now
a capture you orbit, with the transport, share, space info and shortcuts around
it — and that is the whole prototype.

Everything downstream is still in the file and is now **unreachable**: the info
card, the floor rail, the section-cut x-ray, `selectPOI` / `closeCard` /
`enterInterior`, both interior scenes, walk / dollhouse / floorplan, measuring.
`shootPreviews()` is defined but no longer called — it rendered a still per
interior for cards nobody can open.

That is a lot of dormant code. It was left rather than deleted because removing
it is a much larger change than removing the pins, and because whatever restores
entry — clicking the houses directly, a list, a search — will want it. If entry
is not coming back, delete the lot rather than leaving it to rot.

The interior keeps its two captions — `#walkHint` in walk mode and
`#measureHint` while the measure tool is armed. Those are mode feedback, not
onboarding, which is why they survived.

### The auxiliary menu

The aerial overflow (`#moreMenuAerial`) is four items: Space info, Shortcuts,
Help, Terms. Two work — Space info opens `#spaceInfo`, Shortcuts opens the
centred `#shortcuts` sheet. Help and Terms carry `.amx`, which is the class for
an entry that closes the menu and swallows the click, so it feels pressed
without pretending to work.

`#spaceInfo` is a right-edge slide-in rather than a centred dialog: it is a
record you read *while* looking at the capture, so it takes a slice of the
screen, stays translucent, and leaves the scene under it live — the orbit keeps
running and pins stay clickable. Content is static (address, presented by), since
each prototype is one capture of one place. Escape closes it ahead of anything
else, which works only because that guard sits in the **first-registered**
keydown listener; the older `#shortcuts` Escape listener is registered last and
its "before anything else reads the key" comment is wrong about itself.

Both panels sit at `z-index:60`, the same as `#veil`, and `#veil` is later in the
DOM — so anything open there would paint *under* the transition fade.
`enterInterior()` closes them on the way in rather than relying on stacking.

It used to carry Search, Extras, Views, Tutorial, VR and Full screen as well,
copied from the shipping viewer. They are gone: a menu of placeholders pulls
review discussion onto the placeholders. Full screen went with them even though
it worked, which is also why nothing calls `requestFullscreen()` any more.

The interior menu (`#moreMenu`) is a different thing — task controls for the
tour, not auxiliary items — and is untouched by this.

### Screen-space framing

Focus functions size the shot from the field of view rather than magic distances.
The pattern: decide what fraction of the viewport the subject should occupy
(`MULTI_SHARE = 0.55`, `FLOOR_SHARE = 0.78`), solve for radius, then push the
look-at point along screen-right so the subject clears the info card:

```js
const off = (1 - SHARE) * radius * Math.tan(hfov*0.5);
target.x += Math.cos(theta) * off;
target.z -= Math.sin(theta) * off;
```

**Aim at the subject, not at `p.frame`.** `p.frame` is a framing anchor that sits
in front of the house; it was tuned when the floor camera was near-level, where
the difference is invisible. Tilted down, a point further from the camera
projects higher in frame, so aiming short throws the subject up past the top
edge. Use `g.getWorldPosition(c)`.

### Section cuts (the x-ray)

Selecting a house makes it solid and hoverable. Hovering a floor previews it
translucent. Committing to a floor **cuts** the house at that floor's ceiling:
no roof, nothing above.

Three pieces of state, deliberately separate:

- `ghosted` — which structure is active (independent of how it looks, so it stays
  raycastable while solid)
- `xrayOn` — translucent preview
- `sectionOn` — committed cut

`claimMaterials(g, on)` clones that structure's materials so it can be styled
without touching the shared palettes, and hands them back on release.
`restyle()` writes opacity and clipping onto the clones.

Clipping is **per-material** (`renderer.localClippingEnabled` +
`material.clippingPlanes`), not global, so only the selected structure is cut.
The plane keeps `y <= constant`:

```js
new THREE.Plane(new THREE.Vector3(0, -1, 0), constant)
```

`CUT_EPS = 0.08` drops the plane just under the ceiling. Without it, the top
floor lands the plane exactly on the wall cap, the eave and the roof base at
once — and a face coplanar with a clip plane flickers per pixel as its
interpolated distance rounds either side of zero.

Cut houses get **floor plates** (visible only for the cut floor) and **interior
partitions**, or the section reveals an empty shell. Partitions stop short of the
slab above so their tops clear the plane. They have no door openings; real
circulation would need CSG.

`side = DoubleSide` while sectioned, so you see the inside faces of the walls.

Floor lists run **high to low** (Floor 2 above Floor 1) so moving down the list
moves down the house and hover highlighting doesn't invert.

### The ground stack

Site layers are near-coplanar and viewed from 200+ units away, which makes them
fight for the depth buffer — that was the road flicker. Three defences, all
needed:

1. `logarithmicDepthBuffer: true`
2. Named heights in `Y` spaced ~10x further apart than they look
   (`lawn .16`, `lot .34`, `road .52`, `stripe .60`, `walk .68`)
3. Explicit `polygonOffset` per decal material

`M.water` is deliberately excluded from the offset list — it is 3D geometry
(fountain bowls, pool volumes), and slope-scaled offset on curved surfaces makes
them punch through whatever contains them.

Anything new placed on the ground picks a `Y.*` height. Never eyeball it.

### The capture blob

The capture is not a rectangle. `CAPTURE_R = 285` is a mean radius perturbed into
an organic contour, mimicking a real splat boundary. Site content reaches ~230,
so the margin is deliberate. Outside it, flat Google-Earth-style context tiles
fade out. On load the whole thing reveals from the center outward.

`startReveal()` collects its targets **at call time**, not at definition time —
trees and cars are added after the block that defines it, and collecting early
made them pop in at full size.

## Brand

Homes.com orange `#FF7A10`. POI pins are kite/arrow markers: neutral white at
rest, orange on hover and when active. The label sits above the kite.

## Verification

Bash runs in a Linux VM and only sees folders mounted at session start. If the
repo is mounted, syntax-check with:

```bash
mkdir -p ~/chk
python3 - <<'EOF'
import re, pathlib, subprocess, os
out = pathlib.Path(os.path.expanduser('~/chk/chk.mjs'))
for p in sorted(pathlib.Path('.').rglob('index.html')):
    for i, m in enumerate(re.finditer(r'<script(?![^>]*\bsrc=)(?![^>]*importmap)[^>]*>(.*?)</script>', p.read_text(), re.S)):
        out.write_text(m.group(1))
        r = subprocess.run(['node','--check',str(out)], capture_output=True, text=True)
        print(p, i, 'PASS' if r.returncode == 0 else r.stderr)
EOF
```

Write the scratch file under `~`, not `/tmp` — `/tmp` is not writable in the VM.

If bash cannot see the repo, say so rather than claiming the work is verified.
Reading an edit back is not the same as parsing it.

There is no test suite and no linter. The check above plus opening the file in a
browser is the whole safety net, so keep edits surgical and grep for every call
site before renaming anything.

## Known open items

- `PROTOTYPE-RATIONALE.md` is stale — either sync it or delete it. It also still
  describes the commercial prototypes, which are not in this copy.
- Interior partitions have no door openings.
- `apartment_largefile.glb` sits at the repo root at 33 MB and nothing loads it.
