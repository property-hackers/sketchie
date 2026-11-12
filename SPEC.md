# sketchy.json specification

**Draft 0.1. Status: request for comments.**

`sketchy.json` is an open JSON format for building sketches and floor plans. A file
describes the buildings on a property, their levels, and the spaces, walls, openings
and fixtures on each level. It scales from a single exterior outline to a full floor
plan of a multi-building, multi-tenant property, and it records how the sketch changed
over time.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used as described in
RFC 2119.

## Contents

1. [Design principles](#1-design-principles)
2. [File basics](#2-file-basics)
3. [Document](#3-document)
4. [Geometry](#4-geometry)
5. [Structures and levels](#5-structures-and-levels)
6. [Spaces](#6-spaces)
7. [Units of occupancy](#7-units-of-occupancy)
8. [Walls, openings, features and voids](#8-walls-openings-features-and-voids)
9. [Classifications](#9-classifications)
10. [Common element fields](#10-common-element-fields)
11. [Presentation](#11-presentation)
12. [Results](#12-results)
13. [Extensions](#13-extensions)
14. [Format versioning](#14-format-versioning)
15. [Document revisions](#15-document-revisions)
16. [Measurement profiles](#16-measurement-profiles)
17. [Conformance](#17-conformance)
18. [Open questions](#18-open-questions)

## 1. Design principles

1. **Simple to detailed in one structure.** A bare outline is a complete file. More
   detail is added with optional fields, never with a different shape of file.
2. **Describe the building, then the drawing.** The core says what things are. How
   the sketch is drawn on paper is a separate, optional layer that readers may ignore.
3. **Geometry is the source of truth.** Areas come from shapes. Stored totals are a
   cache.
4. **No measurement rules in the core.** The file records the facts measurement rules
   need. The rules themselves are published separately as profiles.
5. **Stable identity.** Every element has an id that survives edits, so versions can
   be compared and merged.
6. **Ignore what you don't know, keep what you don't own.** Readers tolerate unknown
   data, and editors preserve it.
7. **Vendor- and language-agnostic.** Plain JSON, explicit units, and a
   [data dictionary](DICTIONARY.md) of defined values for every type field.

## 2. File basics

- A file is a single UTF-8 JSON object ([RFC 8259](https://www.rfc-editor.org/rfc/rfc8259)).
  No comments, no trailing commas, no byte-order mark.
- File names SHOULD end in `.sketchy.json`. Readers MUST identify a file by its
  `format` field, not its name.
- Suggested media type: `application/vnd.property-hackers.sketchy+json`.
- Keys are `snake_case`. Object key order has no meaning. Array order has no meaning
  unless this spec says it does (polygon points, history and drawing order do).
- Timestamps are [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) strings in UTC,
  for example `"2026-11-12T15:30:00Z"`.
- Writers SHOULD emit numbers without exponents and SHOULD NOT emit `NaN` or
  infinities, which JSON does not allow.

## 3. Document

```json
{
  "format": "sketchy",
  "format_version": "0.1",
  "$schema": "https://property-hackers.github.io/sketchy/schema/0.1/sketchy.schema.json",
  "id": "0193a5f2-6c1e-7a40-9d3e-2b8f4c1a9e70",
  "revision": { "id": "0193a5f8-...", "parents": [], "created_at": "2026-11-12T15:30:00Z" },
  "length_unit": "ft",
  "structures": []
}
```

| Field | Required | Meaning |
|---|---|---|
| `format` | yes | Always `"sketchy"`. |
| `format_version` | yes | Spec version the file follows. See [§14](#14-format-versioning). |
| `$schema` | no | URL of the JSON Schema for that version, for editors and validators. |
| `extensions_used` | no | Extension namespaces present in the file. See [§13](#13-extensions). |
| `extensions_required` | no | Extension namespaces a reader must understand to use the file correctly. |
| `id` | yes | Identity of the sketch. It stays the same across every revision. |
| `revision` | no | This version of the sketch. See [§15](#15-document-revisions). |
| `history` | no | Summaries of earlier revisions. See [§15](#15-document-revisions). |
| `derived_from` | no | `{ "document", "revision" }` this sketch was copied or imported from. |
| `title` | no | Human-readable name. |
| `generator` | no | `{ "name", "version" }` of the program that wrote this revision. |
| `length_unit` | yes | `"ft"`, `"in"`, `"m"`, `"cm"` or `"mm"`. Every length and coordinate uses it. Areas use its square. |
| `measured_to` | no | Default boundary spaces are drawn to. See [§6](#6-spaces). |
| `property` | no | What the sketch is of: `address` (string) and `identifiers` (`[{ "scheme", "value" }]`). |
| `site` | no | `georeference` and `spaces` that belong to no structure (patios, pools, parking). |
| `units` | no | Units of occupancy. See [§7](#7-units-of-occupancy). |
| `structures` | yes | Buildings. May be empty only for a site-only sketch. |
| `classification_schemes` | no | Code tables used by classifications. See [§9](#9-classifications). |
| `definitions` | no | Reusable drawings such as symbols. See [§11](#11-presentation). |
| `presentation` | no | How the sketch is drawn. See [§11](#11-presentation). |
| `results` | no | Stored totals. See [§12](#12-results). |
| `attributes`, `notes`, `extras` | no | See [§10](#10-common-element-fields). |

`site.georeference` places the drawing on the earth:

```json
"georeference": { "crs": "EPSG:4326", "origin": [-84.388, 33.749], "bearing": 0,
                  "control_points": [ { "local": [0, 0], "world": [-84.388, 33.749] } ] }
```

`origin` is the world position of site coordinate `[0, 0]`. `bearing` is the clockwise
angle in degrees from true north to the site's +y axis. `control_points` MAY be given
instead of, or as well as, `origin` and `bearing`, when the drawing was registered
against known points.

## 4. Geometry

### 4.1 Units and frames

- Every coordinate and length is a JSON number in the document's `length_unit`.
  Producers SHOULD round to a precision that suits the unit (for example 0.001 ft or
  0.1 mm). Consumers MUST NOT assume whole numbers.
- Angles are in degrees, counter-clockwise positive, with 0 along +x. (`bearing` in a
  georeference is the one compass-style exception.)
- The **site frame** is a flat plane with +x to the right and +y up. Each structure has
  its own frame, placed in the site frame by its `position` and `rotation`. All levels
  of a structure share that frame, so stacked levels line up. Software that draws with
  +y down flips the axis when rendering.
- A **point** is `[x, y]`.

### 4.2 Paths and curved edges

Outlines are lists of points. Any edge can be a circular arc, using the **bulge**
convention long used by CAD interchange formats: one number per edge, equal to the
tangent of a quarter of the arc's included angle. `0` is a straight edge, `1` is a
half-circle, and positive values bow to the left of the edge's direction of travel.
A bulge can represent any arc up to a full circle except exactly 360°.

The arc's sagitta (the rise of the arc above its chord) is `bulge × chord / 2`.
Producers that store a rise or an angle convert to bulge.

### 4.3 Shapes

Every shape is an object with a `type`.

```json
{ "type": "polygon",  "points": [[0,0],[20,0],[20,10],[0,10]] }
{ "type": "polygon",  "points": [[0,0],[20,0],[20,10],[0,10]], "bulges": [0, 0.3, 0, 0] }
{ "type": "polygon",  "points": [[0,0],[30,0],[30,30],[0,30]],
                      "holes": [ { "points": [[10,10],[20,10],[20,20],[10,20]] } ] }
{ "type": "circle",   "center": [5, 5], "radius": 5 }
{ "type": "ellipse",  "center": [5, 5], "radius_x": 6, "radius_y": 3, "rotation": 30 }
{ "type": "polyline", "points": [[0,0],[12,0],[12,8]], "bulges": [0, 0] }
{ "type": "arc",      "center": [0, 0], "radius": 4, "start_angle": 0, "end_angle": 90 }
```

| Type | Closed | Notes |
|---|---|---|
| `polygon` | yes | `points` lists each vertex once; the first is not repeated. Edge `i` runs from `points[i]` to the next point, and the last edge closes to `points[0]`. Optional `bulges` has one value per edge. Optional `holes` are rings (each `{ "points", "bulges"? }`) cut out of the polygon. Winding order has no meaning. Edges MUST NOT cross. |
| `circle` | yes | |
| `ellipse` | yes | `rotation` turns the x radius away from +x. |
| `polyline` | no | An open path. Optional `bulges` has one value per segment (one fewer than points). |
| `arc` | no | Runs counter-clockwise from `start_angle` to `end_angle`. |

Closed shapes have an area. Open shapes do not. A space whose outline is a `polyline`
is unfinished: the drawing is kept, but it MUST NOT contribute to area totals.

## 5. Structures and levels

A **structure** is a building or other improvement.

| Field | Meaning |
|---|---|
| `id` | Required. |
| `name` | For example `"Main house"` or `"Building B"`. |
| `kind` | From the [dictionary](DICTIONARY.md#structurekind), for example `dwelling`, `accessory_dwelling`, `multifamily`, `garage`, `commercial`. |
| `attachment` | `detached`, `attached` or `semi_detached`, relative to other structures. |
| `position` | Where the structure's `[0, 0]` sits in the site frame. Default `[0, 0]`. |
| `rotation` | Rotation of the structure frame within the site frame. Default `0`. |
| `levels` | Required. |

A **level** is one floor of a structure.

| Field | Meaning |
|---|---|
| `id` | Required. |
| `name` | For example `"First floor"`, `"Basement"`, `"Level 7"`. |
| `grade` | Required. `above`, `below` or `partially_below`, relative to the ground at the structure. |
| `sequence` | Required. Counts up from `1` above grade and down from `1` below grade: first floor `above`/1, second floor `above`/2, basement `below`/1. |
| `elevation` | Optional floor height relative to the structure's ground reference. Orders split levels and mezzanines that share a sequence. |
| `ceiling_height` | Default ceiling height for spaces on the level. |
| `spaces`, `walls`, `openings`, `features`, `voids` | Contents. |

A sketch of part of a building, such as one condo unit, includes only the structure and
level it needs. It does not need to draw the rest of the building.

## 6. Spaces

A **space** is a region of floor with a shape and a use. A whole-floor outline, a garage,
a porch, a single room and a tenant suite are all spaces.

| Field | Meaning |
|---|---|
| `id` | Required. |
| `name` | For example `"Kitchen"` or `"Suite 101"`. |
| `shape` | Required. Any shape from [§4.3](#43-shapes). |
| `use` | What the space is. Recommended values are below. |
| `finished` | Finished to the standard of the building's occupied areas. Omitted means unknown. |
| `enclosure` | `interior`, `enclosed`, `screened`, `covered` or `open`. Distinguishes a room from a three-season room, a screened or covered porch and a deck. Omitted means `interior` for room-like uses. |
| `within` | `id` of the space this one subdivides. |
| `subtract` | `true` if this space's area is deducted from its `within` parent rather than being part of it. |
| `area_factor` | Multiplier applied to this space's area by producers that weight areas. Default `1`. |
| `declared_area` | An area entered by hand rather than computed from the shape, with `reason`. Geometry still wins when a reader computes. |
| `unit` | `id` of the [unit](#7-units-of-occupancy) the space belongs to. |
| `measured_to` | `exterior_face`, `interior_face`, `centerline` or `unspecified`. Overrides the document default. |
| `ceiling_height`, `ceiling_sloped` | Overrides the level default. `ceiling_sloped` means part of the ceiling is lower. |
| `height` | Height of the space, where it differs from the ceiling (for example a two-storey room). |
| `access` | How the space is reached, if it matters to a rule: `interior`, `through_unfinished`, `exterior_only`. |

`use` takes a value from the [dictionary](DICTIONARY.md#spaceuse). It covers whole
areas (`floor`, `unit`, `living`, `common`, `tenant`), rooms (`bedroom`, `kitchen`, …),
vehicle space (`garage`, `carport`, `parking`), outdoor space (`porch`, `deck`, `patio`,
`balcony`, …) and commercial space (`retail`, `lobby`, `circulation`, …).

**Detail levels.** Producers draw what they know:

- *Outline only:* one space per level. The most common simple sketch.
- *Outline and rooms:* rooms set `within` to the outline.
- *Rooms only:* rooms with no `within`.

**How spaces combine.** Unless a measurement profile says otherwise, the area of a
level is the sum of its top-level spaces (those with no `within`), each multiplied by
its `area_factor`. A subdivision gives detail, not extra area. A subdivision with
`subtract: true` is deducted from its parent instead, which covers cut-outs drawn as
their own shapes. Subdivisions MUST lie inside their parent. Their areas need not add up
to the parent's, because walls take space.

## 7. Units of occupancy

A **unit** is a separately occupied part of the property: a dwelling, an accessory
dwelling unit, a condo or a tenant suite. Units cut across structures and levels, and
spaces join a unit with their `unit` field.

| Field | Meaning |
|---|---|
| `id` | Required. |
| `name` | |
| `kind` | From the [dictionary](DICTIONARY.md#unitkind): `dwelling`, `tenant_space`, `common_area`, `storage_unit`, `parking_space`, `other`. |
| `accessory` | `true` for a unit subordinate to a primary dwelling. |

## 8. Walls, openings, features and voids

All of these are optional detail on a level.

### 8.1 Walls

```json
{ "id": "w-north", "start": [0, 9], "end": [10, 9], "bulge": 0, "thickness": 0.5,
  "kind": "exterior", "alignment": "center",
  "bounds": [ { "space": "living", "edge": 2 } ] }
```

| Field | Meaning |
|---|---|
| `start`, `end` | Required. |
| `bulge` | Curves the wall into an arc ([§4.2](#42-paths-and-curved-edges)). Default `0`. |
| `thickness` | Wall thickness. |
| `kind` | From the [dictionary](DICTIONARY.md#wallkind): `exterior`, `interior`, `party`, `half_wall`, `retaining`, `other`. |
| `alignment` | Which line `start`–`end` describes: `center`, `left` or `right` face, looking from `start` to `end`. Default `center`. |
| `height` | Wall height, where it differs from the ceiling. |
| `bounds` | Which space edges this wall forms, as `{ "space", "edge" }` pairs. Lets editors move a wall and its rooms together. |

### 8.2 Openings

Doors, windows and other gaps sit on a wall, or on a space edge when the sketch has no
walls.

```json
{ "id": "d-entry", "kind": "door", "wall": "w-north", "offset": 4.0, "width": 3,
  "swing": "in_left" }
{ "id": "storefront", "kind": "door", "space": "s101", "edge": 0, "offset": 18, "width": 6 }
```

| Field | Meaning |
|---|---|
| `kind` | From the [dictionary](DICTIONARY.md#openingkind), for example `door`, `sliding_door`, `garage_door`, `window`, `bay_window`, `cased_opening`. |
| `wall`, or `space` + `edge` | Where the opening sits. |
| `offset` | Distance along the wall or edge, from its start, to the near side of the opening. On a curved wall it is measured along the arc. |
| `width`, `height`, `sill_height` | Size and placement. |
| `swing` | `in_left`, `in_right`, `out_left`, `out_right` or `none`. |
| `material` | Open string. |
| `projection` | For bay and bow windows, a closed shape for the floor the window projects past the wall. |

### 8.3 Features

Fixed items on a level: stairs, elevators, fireplaces, plumbing fixtures, appliances.

```json
{ "id": "stair-1", "kind": "stair", "position": [12, 4], "rotation": 90,
  "width": 3.5, "length": 12, "connects_to": "l2", "counted_on": "l1", "finished": true }
```

`position` is the centre. `width` runs along local x and `length` along local y, before
`rotation`. Stairs MAY name the level they lead to (`connects_to`) and the level whose
area they count toward (`counted_on`). `kind` comes from the
[dictionary](DICTIONARY.md#featurekind), for example `stair`, `elevator`, `fireplace`,
`sink`, `range` or `island`.

### 8.4 Voids

A **void** is part of a level with no floor: an open-to-below foyer, a stairwell
opening, an atrium.

```json
{ "id": "foyer-open", "name": "Open to below",
  "shape": { "type": "polygon", "points": [[0,0],[12,0],[12,10],[0,10]] } }
```

Voids describe the building. A space with `subtract: true` describes how a producer
chose to deduct an area. Either may be used.

## 9. Classifications

`use` gives a neutral meaning. **Classifications** attach codes from any scheme, so one
file can carry a program's own area codes, a measurement standard's categories and a
client's codes side by side.

```json
"classifications": [
  { "scheme": "org.example.area-codes", "code": "GLA1" },
  { "scheme": "org.example.asset-register", "code": "B-07" }
]
```

`scheme` is a reverse-domain name controlled by whoever defines the codes. Any element
MAY carry `classifications`.

A document MAY include the code tables it uses, so a reader can show names and apply
defaults without knowing the scheme in advance:

```json
"classification_schemes": [
  { "scheme": "org.example.area-codes", "name": "Example area codes",
    "codes": [
      { "code": "GLA1", "label": "First floor living area", "use": "living", "finished": true, "grade": "above" },
      { "code": "GAR",  "label": "Garage", "use": "garage", "finished": false, "area_factor": 1 }
    ] }
]
```

Code defaults never override values set on an element.

## 10. Common element fields

Every element with an `id` (structure, level, space, unit, wall, opening, feature,
void, annotation) MAY carry these:

| Field | Meaning |
|---|---|
| `name` | Display name. |
| `classifications` | [§9](#9-classifications). |
| `attributes` | User-entered key–value pairs shown to people, for example `{ "Flooring": "Oak" }`. String keys and string values. |
| `notes` | Comments: `[{ "text", "author", "created_at" }]`. |
| `source` | Where the element came from: `{ "method", "device", "accuracy", "captured_at", "reference" }`. `method` is one of `drawn`, `measured`, `scanned`, `keyed`, `imported`, `inferred`. `accuracy` is a length. `reference` points to the original data, such as a scan id. |
| `extras` | Machine-readable extension data. See [§13](#13-extensions). |

**Identifiers.** Every `id` MUST be unique within the document. Ids SHOULD be UUIDs,
preferably version 7, which sort by creation time. An element keeps its id for its whole
life, including across revisions. A copied element gets a new id. Ids of deleted
elements SHOULD NOT be reused.

**References** (`within`, `unit`, `wall`, `space`, `connects_to`, `counted_on`, `bounds`,
scopes, styles, layers, definitions) MUST name an id in the same document.

## 11. Presentation

Presentation is how the sketch is drawn. A reader that only needs areas can ignore all
of it. Everything here is keyed to element ids, so the drawing never changes what the
building is.

```json
"presentation": {
  "layers": [ { "id": "lay-dims", "name": "Dimensions", "visible": true, "locked": false } ],
  "styles": { "outline": { "stroke": "#1f2933", "stroke_width": 0.05, "fill": "#e8eef5" } },
  "element_styles": { "main": "outline" },
  "element_layers": { "a-dim-1": "lay-dims" },
  "pages": [],
  "annotations": [],
  "underlays": []
}
```

**Layers** group elements for visibility and locking. A layer MAY name a `parent` layer
to form groups. Elements are assigned with `element_layers`.

**Styles** are named sets of `stroke`, `stroke_width`, `stroke_dash`, `stroke_opacity`,
`fill`, `fill_pattern`, `fill_opacity`, `font_family`, `font_size`, `font_weight`
(`normal`/`bold`), `font_style` (`normal`/`italic`), `text_decoration`
(`none`/`underline`) and `text_color`. Colours are `#rrggbb`. Sizes are in the
document's length unit at drawing scale. `element_styles` maps element ids to styles.

**Pages** lay the sketch out for printing:

```json
{ "id": "p1", "name": "Floor plan", "size": "letter", "orientation": "landscape",
  "scale": 0.0104, "views": [ { "structure": "house", "levels": ["l1"], "offset": [0, 0] } ] }
```

`scale` is paper length over real length in the same unit (`0.0104` is about ⅛ inch to
the foot). `size` is `letter`, `legal`, `tabloid`, `a4`, `a3`, or `[width, height]` in
millimetres. Each view places one structure's levels on the page.

**Annotations** are marks that exist only in the drawing. Each belongs to a `level`, or
to the site when `level` is omitted. Array order is drawing order.

| `kind` | Fields |
|---|---|
| `text` | `position`, `text`, `rotation`, `align` (`left`/`center`/`right`) |
| `label` | `target` (an element id), `fields` (any of `name`, `code`, `area`, `perimeter`, `dimensions`, `factor`), `position`, `rotation`, `scale`. Shows live values of the target. |
| `dimension` | `points` (two or more, for a chain), `offset` (distance from the measured line, positive to the left), `show_total`, `target` (optional wall or space edge it follows, as `{ "wall" }` or `{ "space", "edge" }`), `text` (optional override) |
| `shape` | `shape` (any shape from [§4.3](#43-shapes)): free lines, arcs, circles, ellipses and filled regions |
| `point` | `position` |
| `symbol` | `definition` (an id in `definitions`) or `ref` (`{ "scheme", "name" }` for a library symbol the reader may have), `position`, `rotation`, `scale` (`[sx, sy]`), `mirror` (`x`/`y`/`none`), `host` (optional `{ "wall", "offset", "side" }` for symbols that sit on a wall) |
| `north_arrow` | `position`, `rotation` |

Any annotation MAY set `style`.

**Definitions** are reusable drawings, such as symbols, kept at the document root so
any annotation can place them:

```json
"definitions": [
  { "id": "def-range", "name": "Range", "category": "Kitchen",
    "elements": [ { "kind": "shape", "shape": { "type": "polygon", "points": [[0,0],[2.5,0],[2.5,2],[0,2]] } },
                  { "kind": "shape", "shape": { "type": "circle", "center": [0.7, 0.6], "radius": 0.3 } } ],
    "insertion_point": [1.25, 1] }
]
```

`elements` uses annotation kinds and MAY place other definitions, which allows nesting.
Definitions MUST NOT place themselves, directly or indirectly.

**Underlays** are reference images drawn beneath the sketch, such as a scanned plan or
an aerial photo:

```json
{ "id": "u1", "level": "l1", "uri": "https://example.org/plan.png", "media_type": "image/png",
  "position": [0, 0], "rotation": 0, "units_per_pixel": 0.05, "opacity": 0.5,
  "mirror": "none", "visible": true }
```

`uri` MAY be a `data:` URI to embed the image. Embedded images make files large; see
[open question 3](#18-open-questions).

## 12. Results

`results` stores totals a producer computed, so simple readers can show numbers without
a geometry engine:

```json
"results": [
  { "profile": "org.example.residential-living-area",
    "revision": "0193a5f8-...",
    "computed_at": "2026-11-12T15:30:00Z",
    "generator": { "name": "Example Sketcher", "version": "2.4.0" },
    "values": [
      { "scope": { "unit": "main" }, "measure": "finished_above_grade", "value": 2400 },
      { "scope": { "structure": "house" }, "measure": "garage", "value": 528 }
    ],
    "diagnostics": [ { "severity": "warning", "element": "bonus-room", "message": "Ceiling below 7 ft; excluded." } ] }
]
```

- `profile` names the measurement rules that produced the values. See [§16](#16-measurement-profiles).
- `revision` is the document revision the values were computed for.
- `scope` names any of `structure`, `level`, `unit` or `space`. An empty scope means the
  whole document. `measure` names come from the profile. Values are in `length_unit`
  squared unless the profile says otherwise.
- Results are a cache. If they disagree with the geometry, the geometry wins. A program
  that changes geometry MUST recompute or remove results whose `revision` no longer
  matches.

## 13. Extensions

Any object MAY carry `extras`, a map of extension data with namespaced keys:

```json
"extras": { "org.example.sketcher.snap_grid": 0.5 }
```

- Keys SHOULD be `namespace.field`, where the namespace is a reverse-domain name the
  producer controls. `org.example` is reserved for documentation.
- An extension MUST NOT change the meaning of a core field.
- A document SHOULD list every namespace it uses in `extensions_used`.
- If a reader can't use a file correctly without understanding an extension, the
  writer MUST list that namespace in `extensions_required`. A reader that doesn't
  support every required extension MUST refuse to use the file, or open it read-only
  with a clear warning.
- Editors MUST preserve `extras` they don't understand.
- Extensions that prove widely useful are candidates for the core in a later minor
  version, under a core name.

## 14. Format versioning

These rules follow long experience with file formats that lasted: tolerant readers,
strict writers, additive change, and an explicit marker for anything a reader must not
ignore.

- `format_version` is `MAJOR.MINOR`.
  - A **minor** version only adds optional fields, values or shape types. Every file
    valid under `x.n` is valid under `x.n+1`, with the same meaning.
  - A **major** version may remove or change meaning. Each major version is published
    with a migration guide.
- Readers MUST reject a major version they don't support. Readers SHOULD read newer
  minor versions of a supported major, ignoring unknown fields.
- Writers MUST write the lowest `format_version` that covers every field they emit.
- Editors MUST NOT write a lower version than the one they read unless they first drop
  the newer data, and SHOULD warn the user before doing so.
- A field name is never reused with a new meaning. Fields are deprecated, and only
  removed in a major version.
- Type fields take values from the [data dictionary](DICTIONARY.md). New values are
  added only in minor versions. A reader treats a value it doesn't know as `other`,
  not as an error. Finer distinctions go in classifications, not new values.
- Unknown keys outside `extras` are reserved for future versions. Readers ignore them,
  editors preserve them when they can, and writers MUST NOT invent them.
- Each published version has a JSON Schema at a stable URL and a set of conformance
  fixtures.

## 15. Document revisions

A sketch changes over time: drawn in the field, corrected in the office, revised after
review. `sketchy.json` borrows the model that version-control systems have refined over
decades: every saved version is a complete snapshot, versions link to their parents,
and content can be verified by hash. It deliberately does not require a change log,
locking server or merge algorithm.

### 15.1 Identity

- `id` identifies the sketch and never changes.
- `revision.id` identifies one saved version. A new revision id is minted every time
  changed content is saved.
- Element ids are stable across revisions ([§10](#10-common-element-fields)), so two
  revisions can be compared element by element.

### 15.2 The revision block

```json
"revision": {
  "id": "0193a5f8-2b9c-7d11-8e40-5a6b7c8d9e0f",
  "parents": ["0193a5f2-6c1e-7a40-9d3e-2b8f4c1a9e70"],
  "sequence": 4,
  "created_at": "2026-11-12T15:30:00Z",
  "author": { "name": "J. Smith", "id": "org.example.users/1182" },
  "generator": { "name": "Example Sketcher", "version": "2.4.0" },
  "message": "Corrected rear deck after site revisit",
  "status": "draft",
  "content_hash": "sha256:9f2c…"
}
```

| Field | Meaning |
|---|---|
| `id` | Required in a revision. A UUID, preferably version 7. |
| `parents` | Revision ids this one was made from. Empty for the first revision, one for an ordinary edit, two or more for a merge of parallel edits. |
| `sequence` | Optional counter that increases along a line of edits, for display ("version 4"). Not unique across branches. |
| `created_at`, `author`, `generator`, `message` | Who saved it, when, with what, and why. |
| `status` | `draft` or `final`. A `final` revision is a record of what was issued. It MUST NOT be changed in place; edits create a new revision whose parent is the final one. |
| `content_hash` | Optional integrity check. See below. |

### 15.3 Snapshots and history

- A file is a complete snapshot of one revision. Storing earlier snapshots is up to the
  system that keeps the files: a folder, a database, a document store or a
  version-control repository.
- `history` MAY carry summaries of ancestor revisions, so a lone file still shows its
  lineage:

  ```json
  "history": [
    { "id": "0193a5f2-…", "parents": [], "created_at": "2026-11-10T18:02:11Z",
      "author": { "name": "A. Field" }, "message": "Field sketch", "status": "draft",
      "content_hash": "sha256:41ab…" }
  ]
  ```

  Entries are revision blocks without content, oldest first. They are informational.
  Readers MUST NOT require them.
- A sketch started from another sketch, such as a template or a prior job, names it in
  `derived_from`. It gets a new `id` and starts its own revision history.

### 15.4 Content hash

`content_hash` lets anyone confirm that a file is exactly the revision that was issued.

1. Take the document and remove the `revision` and `history` members.
2. Serialise the result with the JSON Canonicalization Scheme
   ([RFC 8785](https://www.rfc-editor.org/rfc/rfc8785)).
3. Hash the bytes with SHA-256 and write `"sha256:"` followed by the lowercase hex
   digest.

A reader that recomputes a different hash knows the content changed after the
revision was recorded. Digital signatures are out of scope for 0.1, and can be added as
an extension that signs the content hash.

### 15.5 Comparing and merging

Because ids are stable, a reader can compare two revisions without guessing:

- An element is **added** or **removed** if its id appears in only one revision.
- An element is **changed** if its id appears in both with different content.
- When two people edit the same parent, a three-way merge compares each side against
  the common ancestor. Changes to different elements, or different fields of the same
  element, combine cleanly. Two different changes to the same field are a conflict for a
  person to resolve. Geometry (a whole `shape`, or a wall's `start`, `end` and `bulge`)
  is compared as one value.
- The merged result is a new revision listing both sides in `parents`.

These are recommendations. Programs MAY use any comparison or merge method that
respects element identity.

### 15.6 Clean text diffs

Writers SHOULD produce stable output, so plain-text diff tools and version-control
systems show small, readable changes: keep keys in a consistent order, keep array order
stable between saves, format numbers consistently, and avoid rewriting elements that
did not change.

## 16. Measurement profiles

Residential, commercial and international measurement standards count space
differently, and they change over time. `sketchy.json` keeps their rules out of the file
and records the inputs they need:

| Rules need | Field |
|---|---|
| Above or below ground | level `grade` |
| Finished, and how enclosed | space `finished`, `enclosure` |
| What the space is | space `use`, `classifications` |
| Which dwelling or tenant | space `unit`, unit `accessory` |
| Ceiling height and slope | `ceiling_height`, `ceiling_sloped` |
| Where it was measured to | `measured_to` |
| Openings to below, deductions | `voids`, space `subtract` |
| Weighted areas | space `area_factor` |
| Bay projections | opening `projection` |
| Stairs and which floor counts them | feature `counted_on` |
| How a space is reached | space `access` |

A **profile** is a published set of rules for turning a `sketchy.json` document into
totals. It has a reverse-domain name, defines its `measure` names and rounding, and
lists the fields it reads. Profiles are published separately from this spec, so a new
standard or a revised one never changes the format.

## 17. Conformance

- A **valid document** has every required field, unique ids, resolvable references,
  closed shapes that don't self-intersect, and subdivisions inside their parents.
- A **reader** accepts every valid document of a supported major version and follows
  [§14](#14-format-versioning).
- An **editor** is a reader that also writes. It preserves ids, `extras`, unknown
  fields where possible and `revision` lineage ([§15](#15-document-revisions)).
- A JSON Schema per version checks structure. Rules a schema can't express (unique ids,
  references, geometry) are listed in the conformance fixtures.

## 18. Open questions

1. Should a file be able to mix units, or should producers always convert to one
   `length_unit`?
2. Is `sequence` plus optional `elevation` enough to order every kind of split level?
3. Should a zip bundle be defined for sketches that carry images (underlays, rendered
   previews), instead of `data:` URIs?
4. Which measurement profiles should be documented first?
5. Should three-dimensional detail (roof pitch, wall heights for volume, floor
   elevations) get its own optional layer?
6. Should signatures over `content_hash` be specified in the core or left to
   extensions?
