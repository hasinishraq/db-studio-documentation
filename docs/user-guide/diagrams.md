# Schema diagram

Click the diagram icon next to a schema (or right-click → **Diagram**). Tables are drawn from the
live database with their keys, types and the relationships between them.

![Schema diagram](../assets/screenshots/31-diagram.png){ .shot }

- **Drag** the background to pan, **scroll** to zoom, **drag a table** to move it.
- **Fit** frames every table; **+ / −** zoom.

![Arranged diagram](../assets/screenshots/32-diagram-zoomed.png){ .shot }

## Saving as an image

**Export image** saves the whole diagram, not just the part on screen, as **PNG**, **JPEG** or
**PDF**, always in light colours on white:

![Export image](../assets/screenshots/33-diagram-export.png){ .shot }

The file is drawn in Rust (resvg and svg2pdf, with bundled fonts), so it looks the same on every
machine:

![The exported PNG](../assets/screenshots/34-diagram-export-output.png){ .shot }

!!! note
    Table positions are not saved yet; reopening the tab arranges the tables again.
