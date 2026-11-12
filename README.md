# sketchy.json

**A proposal for an open, vendor-agnostic JSON format for building sketches and floor
plans.**

> **Status: proposal.** Nothing is final. Issues and pull requests are welcome.

## The problem

Sketch and floor-plan programs each save drawings in their own closed format. A sketch
made in one program usually can't be opened in another, so sharing it means exporting
a picture and retyping the numbers. Nobody owns a common format for the most basic
fact about a building: its shape and how much floor area it has.

## The idea

`sketchy.json` is a plain JSON file, saved as `*.sketchy.json`, that describes the
buildings on a property:

- **Structures**: houses, garages, outbuildings, commercial buildings. A property can
  have several.
- **Levels**: floors above and below ground.
- **Spaces**: areas with a shape and a use. One space can be a whole floor outline, a
  room, a porch or a tenant suite.
- **Units**: separately occupied parts, such as a dwelling, an accessory dwelling unit,
  a condo or a tenant space.
- **Optional detail**: walls, doors, windows, stairs, fixtures and openings to the
  floor below.

## Principles

1. **Simple to detailed in one format.** A single exterior outline is a complete,
   valid file. A full floor plan of a multi-storey, multi-tenant building uses the
   same structure with more detail filled in.
2. **Describe the building, not just the drawing.** The core records what things are.
   How the sketch looks on paper (labels, dimensions, styles, pages) is an optional
   layer.
3. **Geometry is the source of truth.** Areas are computed from shapes. Stored totals
   are a convenience that readers can recompute.
4. **No measurement rules built in.** Residential, commercial and international
   measurement standards count space differently. The file records the facts those
   rules need, such as finished or not, above or below ground and ceiling height, and
   the rules live outside the format.
5. **Vendor- and language-agnostic.** Plain JSON with explicit units. No product,
   report form or programming language is assumed.
6. **Extensible without breaking.** Namespaced extension fields carry program-specific
   data, and readers keep data they don't understand.
7. **Curves are first-class.** Any wall or outline edge can be an arc, and shapes
   include circles and ellipses.
8. **Versioned like source code.** Every saved version is a complete snapshot with a
   revision id, links to the versions it came from and an optional content hash.
   Element ids stay stable across versions, so any two versions can be compared and
   merged.

The full draft is in **[SPEC.md](SPEC.md)**.

## A taste

A one-storey house drawn as an exterior outline:

```json
{
  "format": "sketchy",
  "format_version": "0.1",
  "id": "c6a1b9e2-1f0c-4c55-9a4e-6f3f2a1d7b10",
  "length_unit": "ft",
  "structures": [
    {
      "id": "house",
      "kind": "dwelling",
      "levels": [
        {
          "id": "l1",
          "grade": "above",
          "sequence": 1,
          "spaces": [
            {
              "id": "main",
              "use": "living",
              "finished": true,
              "shape": { "type": "polygon", "points": [[0, 0], [40, 0], [40, 30], [0, 30]] }
            }
          ]
        }
      ]
    }
  ]
}
```

## Examples

| File | What it shows |
|---|---|
| [`single-family.sketchy.json`](examples/single-family.sketchy.json) | A two-storey house in feet, drawn as exterior outlines: a curved sunroom, a bay window projection, an open-to-below foyer, a basement split into finished and unfinished parts, an attached garage, a porch and deck, a rotated detached cottage (an accessory unit), a site pool, area codes, symbols, two print pages and a final revision with history. |
| [`condo-unit.sketchy.json`](examples/condo-unit.sketchy.json) | One condo unit on the seventh floor of a larger building, in metres, drawn room by room: party and partition walls, doors and windows placed on walls, fixtures, a balcony with a curved railing, and a sketch derived from a building template. |
| [`commercial-building.sketchy.json`](examples/commercial-building.sketchy.json) | A three-storey retail and office building: tenant suites and common cores within each floor outline, one tenant spread across two spaces, a circular atrium open to below, stairs and elevators, storefront openings placed on space edges without walls, site parking, a georeference, an extension namespace and a revision that merges two parallel edits. |

Addresses, people and codes are invented. Code schemes and profiles use the reserved
`org.example` namespace. Element ids are short and readable for clarity; real files
should use UUIDs.

## Next steps

1. Review the [draft specification](SPEC.md) and examples.
2. Add a JSON Schema and conformance fixtures with expected areas.
3. Document measurement rules as separate, named profiles.
