# Poster Tiler: handoff

## What it is
Poster Tiler (shown in the app as "Plakatiprint") is a single-file web app (`poster-tiler.html`) that splits a large poster into A4 sheets, then prints them or saves them as a PDF. The whole interface and the PDF text are in Estonian. The older claude.ai artifact (https://claude.ai/artifact/Wy1JjaGSV86LLfcEeaWMb7) has not been updated with this version.

## How it works
- **Input:** PNG, JPG or WebP image, or a PDF (for example a Word poster exported to PDF). PDFs are rendered with pdf.js 3.11.174. The worker script is loaded as a `<script>` tag, so pdf.js runs its worker on the main thread.
- **Finished size:** A3, A2, A1 or A0 in portrait or landscape. Custom size takes a width; the height follows the image's aspect ratio. Fit mode is either contain or cover.
- **Extra settings (`Lisaseaded`, closed by default):** orientation, fit mode, unprintable edge in mm, overlap in mm, trim lines, and whether to print the assembly guide. Paper is always A4.
- **Layout:** `layout()` tries both paper orientations and picks the one that needs fewer sheets.
  - Step between sheets = printable width − overlap.
  - Columns = ceil((W − overlap) / step).
- **Preview:** a canvas on a cutting-mat background shows the sheet grid. Overlap bands are tinted magenta, and each sheet is labelled A1, A2, B1… (rows are letters, columns are numbers).
- **Stats:** number of sheets, grid, finished size, and the image's effective dpi with a quality pill (150 dpi or more is good, 100 or more is acceptable).
- **Output:** jsPDF 2.5.1 builds the PDF.
  - The assembly guide can be read in the app under the action buttons. Printing it as a first page (thumbnail and sheet map) is off by default.
  - Each sheet's image is rendered at min(200, effective dpi × 1.25) dpi as JPEG.
  - A magenta dashed trim line sits at overlap/2 on the left and top edges of every sheet except the first column and row.
  - Alignment ticks sit at the matching spot on the right and bottom edges.
  - Each sheet has a label in its bottom margin.
- **Saving:** in the claude.ai artifact the file is saved through `claude.use("downloads").save({filename, data: blob})`, which needs `capabilities: {downloads: true}`. Self-hosted, the page has no `window.claude` and saves through an `<a download>` link.
- **Printing (self-hosted only, hidden in claude.ai):** the PDF is printed by the viewer's browser, so it goes to whatever printer that device can reach. It sets `PrintScaling: None`.
  - Desktop: "Print sheets" loads the PDF in a hidden iframe and calls `print()`.
  - Phones and tablets (`handheld`): "Prepare for printing" builds the PDF, then "Print now" opens the share sheet, which has Print. iOS needs that second tap, and `navigator.share` with files needs HTTPS. Without it the PDF opens in a tab.
  - `handheld` also caps PDF input at 16 MP (iOS blanks larger canvases) and output at 150 dpi.

## Deployment
Runs as a Dockge stack on a host in the tailnet: `compose.yaml` serves `poster-tiler.html` with nginx on `127.0.0.1:8088`. Tailscale Serve publishes it over HTTPS on port 8443 (`sudo tailscale serve --bg --https=8443 8088`). Port 8443 is used because something else already listens on 443 on that host.

To update, copy `poster-tiler.html` to `/opt/stacks/poster-tiler/` and restart the stack in Dockge. `/opt/stacks` is root-owned, so this needs sudo.

## Constraints when run as a claude.ai artifact
- Scripts load only from cdnjs, jsDelivr or unpkg.
- No `window.print()` and no direct downloads.
- Light and dark theme tokens are defined on `:root`.

## Possible next steps
- Let the user set the poster size in sheets (for example 3 × 3 A4).
- Add an option to print only selected sheets.
- Add a scale-check ruler (a 10 cm bar) on each sheet.
