# Symfony UX Frontend Stack

This project uses the Symfony UX frontend stack. Seven agent skills are installed to help you work with it.

## Which tool to use

- **Pure JS behavior, no server round-trip** -- use the `stimulus` skill
- **Navigation, partial page updates** -- use the `turbo` skill
- **Reusable static UI component** -- use the `twig-component` skill
- **Reactive component that re-renders on user input** -- use the `live-component` skill
- **SVG icons (local or Iconify)** -- use the `ux-icons` skill
- **Interactive maps (Leaflet / Google Maps)** -- use the `ux-map` skill
- **Not sure which one fits** -- use the `symfony-ux` skill (orchestrator / decision tree)

## Key rules

- Always render `{{ attributes }}` on the root element of a LiveComponent
- Prefer HTML syntax (`<twig:Alert />`) over Twig syntax (`{% component 'Alert' %}`)
- Use `data-model="debounce(300)|field"` for text inputs in LiveComponents
- Stimulus controllers must clean up listeners and observers in `disconnect()`
- Turbo Frame IDs must match between the page and the server response
- Use Turbo Streams when updating multiple page sections; use Frames for a single section
- `<twig:Turbo:Stream:*>` components (e.g. `<twig:Turbo:Stream:Append target="#comments">`) can replace raw `<turbo-stream>` HTML
- Prefer `<twig:ux:icon name="..." />` over `{{ ux_icon('...') }}` for consistency
- Map containers must have an explicit height (`style="height: 400px;"`)
- Use `fitBoundsToMarkers()` instead of manually calculating center/zoom
- Lock on-demand icons before deploying: `php bin/console ux:icons:lock`
