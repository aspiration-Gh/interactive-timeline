# Interactive Timeline

English | [中文](README.md)

A **double-click-and-use** interactive timeline in a single HTML file. Zero dependencies, no build step, works offline. Lay out a plan with the mouse in the browser and export a PNG for notes or sharing.

![preview](screenshots/preview-dark.png)

## Features

- **Single file**: the whole app is one `index.html` (~38 KB) — no build, no CDN, no backend, works offline
- **Multiple timelines**: switch / create / rename / delete from the top dropdown; data lives in `localStorage` with JSON export
- **Flat structure**: event points plus optional "phase" bands — no parent/child hierarchy
- **Free placement**: each event can sit above or below the axis at any distance; in auto mode they alternate and stagger when dates are close
- **Vertical connectors**: a straight stem drops from the date on the axis to the event box, which is centered on its date
- **Phase dividers**: add any number of dashed markers, name them, color them, drag them to another date
- **Direct editing**: click the axis to insert an event at that date; drag an event sideways to change its date, vertically to flip side and distance; edit title / note / color in the side panel
- **PNG export**: exports the whole timeline at 2x, background follows the current theme
- **Dark / light theme**: one click in the header

## Quick start

1. Download `index.html`
2. Open it in a browser (Chrome / Edge / Firefox)
3. Click the axis to add your first event, then drag things around

Nothing to install.

## Controls

| Action | Effect |
| --- | --- |
| Click the axis | Insert an event at that date |
| Drag event sideways | Change its date (snaps to day) |
| Drag event vertically | Switch side and change distance from the axis |
| Drag a dashed marker | Change that marker's date |
| Drag empty space | Pan |
| Wheel | Zoom around the cursor |
| Double-click empty space | Fit the whole timeline |
| Click event / band / marker | Open its form in the side panel |
| `0` / `+` / `-` / `Esc` | Fit / zoom in / zoom out / deselect |

## Data format

Stored under the `timeline.app.v1` key in `localStorage`, exportable as JSON:

```json
{
  "lists": [
    {
      "id": "abc1234",
      "name": "My timeline",
      "events": [
        { "id": "e1", "title": "Mid-term review", "date": "2026-11-15",
          "note": "", "side": "up", "dist": 90, "color": "#7c8cff" }
      ],
      "phases": [
        { "id": "p1", "name": "Phase 1", "start": "2026-10-18", "end": "2026-11-17",
          "color": "#7c8cff" }
      ],
      "markers": [
        { "id": "m1", "name": "Month end", "date": "2026-11-17", "color": "#f2777a" }
      ]
    }
  ]
}
```

- `side`: `auto` (alternate) / `up` / `down`
- `dist`: distance in pixels from the axis to the event center; only used when `side` is `up` / `down`

## How it works

An SVG canvas plus plain front-end state — no libraries:

```mermaid
flowchart LR
  A[localStorage / imported JSON] --> B[Layout: date to x mapping]
  B --> C[SVG render: axis, ticks, bands, markers, events]
  C --> D[Pointer events: drag / zoom / insert]
  D --> A
  C --> E[Clone SVG + inline styles + canvas 2x]
  E --> F[PNG download]
```

| Step | Approach |
| --- | --- |
| Time mapping | `x = (t - t0) / day * kx`, with `kx` derived from the total span so short spans do not crowd and long spans do not stretch |
| Stacking | explicit `side` uses `dist` directly; `auto` alternates sides and pushes outward on collision |
| Dragging | horizontal delta becomes a date, vertical delta becomes side and distance, committed on pointer-up |
| Zoom | anchor at the cursor by adjusting `translate + scale`, clamped to 0.25–3.2 |
| PNG export | clone the `world` group, inject inline `<style>` with resolved CSS variables, serialize to SVG → `img` → canvas 2x → Blob |

## Limitations

- Data lives in **browser storage**: switching browsers, clearing site data or private mode loses it — export JSON for backup
- Some browsers block storage on `file://`; the app then shows a banner and only JSON backup works
- Day-level precision only, no hours/minutes
- No Markdown import, no multi-lane Gantt view
- Single-file implementation, no automated tests

## License

[MIT](LICENSE)
