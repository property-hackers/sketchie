# sketchy.json data dictionary

**Draft 0.1.** Every enumerated field in the [specification](SPEC.md) takes one of the
values listed here. Values are lowercase `snake_case`.

**Rules**

- A writer MUST use a listed value. When nothing fits, it writes `other` and MAY add a
  [classification](SPEC.md#9-classifications) with the precise meaning.
- New values are added only in minor versions of the spec, never removed or renamed
  within a major version.
- A reader that meets a value it doesn't know (from a newer minor version) MUST treat
  it as `other` for fields that have `other`, and SHOULD warn for fields that don't.
- Finer distinctions (a primary bedroom, a walk-in closet, a fire-rated wall) belong in
  classifications, not in new dictionary values.

## Document and geometry

### `length_unit`
| Value | Meaning |
|---|---|
| `ft` | International foot (0.3048 m) |
| `in` | Inch (0.0254 m) |
| `m` | Metre |
| `cm` | Centimetre |
| `mm` | Millimetre |

### `shape.type`
| Value | Meaning |
|---|---|
| `polygon` | Closed outline with optional curved edges and holes |
| `circle` | Closed circle |
| `ellipse` | Closed ellipse |
| `polyline` | Open path with optional curved segments |
| `arc` | Open circular arc |

### `measured_to`
| Value | Meaning |
|---|---|
| `exterior_face` | Outside face of exterior walls |
| `interior_face` | Inside face of the walls around the space |
| `centerline` | Centre of the surrounding walls |
| `unspecified` | Not known |

### `source.method`
| Value | Meaning |
|---|---|
| `drawn` | Drawn by hand in software, without measurement |
| `measured` | Measured on site, for example with a tape or laser meter |
| `scanned` | Captured by a scanning device |
| `keyed` | Typed in from another record, such as a plan or survey |
| `imported` | Converted from another file format |
| `inferred` | Derived by software from other data |

## Buildings

### `structure.kind`
| Value | Meaning |
|---|---|
| `dwelling` | A house or other primary residence |
| `accessory_dwelling` | A secondary, self-contained dwelling on the same property |
| `multifamily` | A building of several dwelling units |
| `manufactured_home` | A factory-built home |
| `garage` | A building for vehicles |
| `carport` | A roofed, open-sided vehicle shelter |
| `outbuilding` | A shed, workshop, studio or similar secondary building |
| `barn` | An agricultural building |
| `commercial` | A building for retail, office or service use |
| `industrial` | A building for manufacturing, warehousing or distribution |
| `mixed_use` | A building combining residential and commercial uses |
| `parking_structure` | A multi-level parking building |
| `other` | |

### `structure.attachment`
| Value | Meaning |
|---|---|
| `detached` | Stands alone |
| `attached` | Shares a wall or roof with another structure on the property |
| `semi_detached` | Shares one wall with a neighbouring building |

### `level.grade`
| Value | Meaning |
|---|---|
| `above` | The floor is at or above the surrounding ground |
| `partially_below` | Part of the level is below the surrounding ground, such as a walkout basement |
| `below` | The level is entirely below the surrounding ground |

### `unit.kind`
| Value | Meaning |
|---|---|
| `dwelling` | A home: a house, apartment, condo or accessory dwelling |
| `tenant_space` | A leased commercial or industrial space |
| `common_area` | Space shared by occupants, such as a lobby or corridor |
| `storage_unit` | A separately held storage space |
| `parking_space` | A separately held parking space |
| `other` | |

## Spaces

### `space.use`
| Group | Value | Meaning |
|---|---|---|
| Whole areas | `floor` | The whole outline of a level |
| | `unit` | The whole outline of one unit |
| | `living` | General finished living area, when rooms are not drawn |
| | `common` | Shared area of a multi-unit building |
| | `tenant` | Leased area of a commercial building, when not further divided |
| Rooms | `bedroom` | |
| | `bathroom` | |
| | `kitchen` | |
| | `dining` | |
| | `living_room` | |
| | `family_room` | |
| | `office` | A room or area used as an office |
| | `laundry` | |
| | `closet` | |
| | `hall` | |
| | `foyer` | |
| | `stair` | Floor area occupied by a stair |
| | `utility` | Utility or mechanical room |
| | `storage` | Storage area or room |
| | `bonus_room` | A flexible room, such as over a garage |
| Vehicles | `garage` | Enclosed vehicle space |
| | `carport` | Roofed, open-sided vehicle space |
| | `parking` | Parking area or stall |
| Outdoor | `porch` | Roofed area attached to a building entrance |
| | `deck` | Raised platform, usually unroofed |
| | `patio` | Paved area at ground level |
| | `balcony` | Platform projecting from an upper level |
| | `terrace` | Large open area on a roof or level |
| | `breezeway` | Roofed passage connecting structures |
| | `pool` | Swimming pool |
| Commercial | `retail` | |
| | `restaurant` | |
| | `warehouse` | |
| | `industrial` | Manufacturing or production area |
| | `lobby` | |
| | `circulation` | Corridors and other shared routes |
| | `amenity` | Shared amenity, such as a fitness room |
| | `loading` | Loading dock or bay |
| | `other` | |

### `space.enclosure`
| Value | Meaning |
|---|---|
| `interior` | Part of the building's interior, protected for year-round use. The default for room-like uses |
| `enclosed` | Walled and roofed, but not part of the interior, such as a three-season room |
| `screened` | Roofed with screened sides |
| `covered` | Roofed, open sides |
| `open` | No roof |

### `space.access`
| Value | Meaning |
|---|---|
| `interior` | Reached from finished interior space |
| `through_unfinished` | Reached only through unfinished space |
| `exterior_only` | Reached only from outside |

## Walls, openings and features

### `wall.kind`
| Value | Meaning |
|---|---|
| `exterior` | Separates the building from outside |
| `interior` | Separates spaces inside the building |
| `party` | Shared with another unit or building |
| `half_wall` | A low or partial-height wall, such as a railing wall |
| `retaining` | Holds back earth |
| `other` | |

### `wall.alignment`
| Value | Meaning |
|---|---|
| `center` | `start`–`end` is the wall's centreline |
| `left` | `start`–`end` is the wall's left face, looking from `start` to `end` |
| `right` | `start`–`end` is the wall's right face |

### `opening.kind`
| Value | Meaning |
|---|---|
| `door` | Hinged single door |
| `double_door` | Hinged pair of doors |
| `sliding_door` | |
| `pocket_door` | |
| `bifold_door` | |
| `garage_door` | Overhead or sectional vehicle door |
| `window` | |
| `bay_window` | Window projecting in straight segments |
| `bow_window` | Window projecting in a curve |
| `skylight` | Roof window |
| `cased_opening` | Framed opening with no door |
| `pass_through` | Opening between rooms above floor level |
| `other` | |

### `opening.swing`
| Value | Meaning |
|---|---|
| `in_left` | Opens into the space the opening is placed from, hinged on the left |
| `in_right` | Opens inward, hinged on the right |
| `out_left` | Opens outward, hinged on the left |
| `out_right` | Opens outward, hinged on the right |
| `none` | Does not swing |

### `feature.kind`
| Group | Values |
|---|---|
| Circulation | `stair`, `ramp`, `elevator`, `escalator` |
| Structure | `column`, `fireplace`, `chimney` |
| Building systems | `water_heater`, `furnace`, `hvac_unit`, `electrical_panel` |
| Plumbing | `sink`, `toilet`, `tub`, `shower` |
| Kitchen and laundry | `range`, `refrigerator`, `dishwasher`, `island`, `washer`, `dryer` |
| | `other` |

## Revisions and results

### `revision.status`
| Value | Meaning |
|---|---|
| `draft` | Work in progress |
| `final` | Issued. Must not be changed in place |

### `diagnostic.severity`
| Value | Meaning |
|---|---|
| `info` | |
| `warning` | |
| `error` | |

## Presentation

| Field | Values |
|---|---|
| `annotation.kind` | `text`, `label`, `dimension`, `shape`, `point`, `symbol`, `north_arrow` |
| `label.fields` | `name`, `code`, `area`, `perimeter`, `dimensions`, `factor` |
| `page.size` | `letter`, `legal`, `tabloid`, `a4`, `a3`, or `[width, height]` in millimetres |
| `page.orientation` | `portrait`, `landscape` |
| `text.align` | `left`, `center`, `right` |
| `mirror` | `none`, `x`, `y` |
| `font_weight` | `normal`, `bold` |
| `font_style` | `normal`, `italic` |
| `text_decoration` | `none`, `underline` |
