# Prototype rationale

Four prototypes, two strategies against two verticals. The hi-fidelity build (`homes/a-hifi`) is excluded — it is a rendering test of Strategy A, not a separate concept.

|  | Commercial (LoopNet) | Residential (Homes.com) |
|---|---|---|
| **A — Pin based** | `commercial/a` · 5 pins → 4 interiors | `homes/a` · 5 pins → 3 interiors |
| **B — Simplified** | `commercial/b` · 1 entry → lobby | `homes/b` · 1 entry → great room |

All four share the same spine: an orbiting aerial of the property, a transition that fades rather than cuts, and an interior tour with walk / dollhouse / floorplan views and a measuring tool. The fork is **how many front doors the exterior offers**.

---

## Commercial A — pin based

**What problem does it solve?**

A commercial asset is not one space. This campus has a main entrance, a central courtyard, an amenity pavilion, a parking structure and a second tower lobby — five things a tenant evaluates separately, mapping to four distinct interiors. A single tour entry answers "what does it look like inside" while leaving "what is actually here, and where" unanswered. Brokers also merchandise these as separate sale points: available suite, amenity, parking ratio. Separate sale points need separate front doors.

**How does it affect the user?**

They read the property before committing to a tour. Five numbered markers name what exists; each card carries a real interior still shot from the actual 3D scene, so the preview is not a promise. They can jump straight to the space that matters, or use prev/next to be walked through all five in authored order. Because the camera eases to each point's own elevation rather than cutting, they learn where things sit relative to each other — they leave with a spatial model of the campus, not a list of rooms.

**Why is it better than the alternative?**

Strategy B cannot express inventory. From B's exterior you cannot tell an amenity pavilion exists, so discovery becomes sequential and depends on the user walking far enough to find it. A also lets different content hang off different spaces — square footage, availability, status — which is the actual commercial sales motion. The cost is real: more chrome on first load, and content to author and maintain per pin.

---

## Commercial B — simplified

**What problem does it solve?**

Scaffolding around nothing. Most listings have exactly one thing worth showing — the lobby, or the single available suite. A pin layer over a single-destination property manufactures choices that do not exist, and every pin is copy somebody has to write and keep true when a suite leases.

**How does it affect the user?**

One button, one decision, the shortest possible path to the interior. The exterior orbit does its job — massing, setting, immediate context — and then gets out of the way. Nothing to interpret, nothing to dismiss, no wondering whether they missed a marker on the far side of the building.

**Why is it better than the alternative?**

When there is one destination, five markers pointing at it is worse than none. B also scales down to thin content: one capture plus an aerial is a complete experience, which matters for the long tail of listings that will never justify per-space authoring. Cheaper to produce and harder to get wrong.

---

## Homes A — pin based

**What problem does it solve?**

New-home communities sell a community, not a house. The buyer's questions are neighborhood-shaped: which homesites are still available, what the clubhouse and pool are like, whether there is a park, where the model sits. A single model-home tour answers one of those and leaves the community invisible.

**How does it affect the user?**

They browse inventory spatially. Homesite 14 and Homesite 22 are not rows in a table — they are places, with a visible relationship to the pool and the pocket park. The card carries the residential decision set: plan name, square footage, bed and bath count, availability. Note that five pins map to only three interiors: two homesites share the great-room tour. That is deliberate, and it is the point — pins here represent **parcels**, not rooms, which is how builders actually sell.

**Why is it better than the alternative?**

B shows one model home well but cannot express availability or amenity, so a buyer comparing homesites has to be told about them somewhere else — and that somewhere else is usually a separate page that breaks the spatial context this view just built. A puts inventory on the map, which is the whole job of a builder's community page. It presumes you have content for each pin, which is a real precondition.

---

## Homes B — simplified

**What problem does it solve?**

The single-listing case: a resale home, or a builder with one model open. Pins would be inventing choices the property does not have.

**How does it affect the user?**

They land, orbit the home in its actual context — lot shape, street, neighbors, setback — and step inside in one click. The aerial does the thing interior photography cannot: it establishes where this house *is*, not just what it contains. Then it hands off.

**Why is it better than the alternative?**

For a single property, A's markers would be five ways into the same house. B also matches the shape of the market — the overwhelming majority of residential inventory is one home with one capture, and a strategy that only works with authored multi-point content does not survive contact with it.

---

## The actual decision

A versus B is not a winner-take-all. It is a **content threshold**:

- **Use A** when the property has two or more genuinely distinct destinations with distinct sales content attached.
- **Use B** below that line.

The two verticals differ in what a pin *means*, which is worth settling before either ships. In commercial, a pin is a **space within one asset**. In residential, a pin is a **parcel within a community**. Same interaction, different unit of inventory — and that difference should drive the card content, not the pin design.

The open risk in both A builds is density. Markers cluster where the inventory clusters — three homes around the cul-de-sac, two lobbies on the same tower face — and that is exactly where a buyer is comparing most carefully.
