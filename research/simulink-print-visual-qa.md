# Verifying the Simulink print workflow for visual QA

**Context:** Research ticket for [felixloose/beautifySimulink#2](https://github.com/felixloose/beautifySimulink/issues/2), supporting the wayfinding map [felixloose/beautifySimulink#1 — "Specify a readable Simulink diagram skill"](https://github.com/felixloose/beautifySimulink/issues/1).

**Branch note:** This is a throwaway research branch (`research/simulink-print-visual-qa`). It is not intended to be merged via PR; it exists only to capture findings for issue #2 per the Wayfinder research-ticket procedure.

**Question investigated:** What supported MATLAB/Simulink command/API sequence can an autonomous agent use to render a Simulink block diagram to an image for visual QA — including output format, layout/print constraints, safe temp-file handling, and limitations to surface (especially for headless/CI use)?

---

## Summary

The only currently documented, supported command/API for rendering a Simulink block diagram to an image file is the MATLAB `print` function used with Simulink's `-s<ModelName>` switch. There is **no** documented `Simulink.BlockDiagram.print` or `Simulink.BlockDiagram.createDiagramObject` function — those names do not appear in official MathWorks docs (`Simulink.BlockDiagram` only exposes structural-editing methods such as `createSubsystem`). The single most important constraint for an autonomous/headless agent: **Simulink printing/export is explicitly turned off when MATLAB runs in no-display mode.** A separately licensed alternative exists via Simulink Report Generator's `slreportgen.report.Diagram` reporter, which defaults to SVG snapshots.

---

## 1. Command/API sequence to render a Simulink diagram to an image/document file

**Primary source:** [Print Models Programmatically](https://www.mathworks.com/help/simulink/ug/print-from-the-matlab-command-line.html) (`mathworks.com/help/simulink/ug/print-from-the-matlab-command-line.html`; fetched via Wayback Machine mirror, `web.archive.org/web/20241006045843/...`, due to sandbox access restrictions on `www.mathworks.com`)

- The documented mechanism is the MATLAB `print` command with Simulink's `-s` switch, which specifies "the system (model or subsystem) to print":
  ```matlab
  openExample('sldemo_enginewc');
  print -sCompression
  ```
- "When you use the `print` command, you can print only one diagram. To print multiple levels in a model hierarchy, use multiple `print` commands, one for each diagram."
- To export to an image format: "To print your model to an image file format such as `.png` or `.jpeg`, use the `-device` argument with the MATLAB `print` command," e.g.:
  ```matlab
  print('-smodel','-djpeg','new_name')
  ```
- Per the [`print` function reference](https://www.mathworks.com/help/matlab/ref/print.html) (`fig` argument, Wayback mirror `web.archive.org/web/20250219063334/...`): *"When specifying a Simulink block diagram, precede the model name with `-s`. Specify the current model using `'-s'`."*
- The model must be **open/loaded** first (`open_system`/`load_system`, or already loaded) — every documented example precedes `print` with `openExample(...)` or `open_system(...)`.
- There is also an interactive-only route (Simulink Editor → **Simulation** tab → **Print** → Print Model dialog), documented at [Print Model Diagrams](https://www.mathworks.com/help/simulink/ug/print-model-diagrams.html) (Wayback mirror `web.archive.org/web/20241205011946/...`) — not usable from a headless/autonomous agent.

## 2. Supported output formats and which to prefer

**Source:** [`print` function reference](https://www.mathworks.com/help/matlab/ref/print.html)

- Raster: `-djpeg` (.jpg), `-dpng` (.png), `-dtiff`/`-dtiffn` (.tif), `-dmeta` (.emf, Windows only).
- Vector: `-dpdf` (.pdf), `-deps`/`-depsc`/`-deps2`/`-depsc2` (.eps), `-dmeta` (.emf), `-dsvg` (.svg).
- **Explicit documented constraint: "You cannot save Simulink block diagrams as EPS files."** EPS is excluded for Simulink specifically, despite being listed generically for figures.
- Doc guidance on tradeoffs: raster images "are widely used by web browsers and other applications that display graphics. However, they do not support transparency or scale well..."; vector "files store commands that redraw the figure. This type of format scales well, but can result in a large file."
- **Recommendation for visual QA: PNG (`-dpng`)** — a fixed-resolution raster that vision models/humans can consume directly, without the EPS incompatibility or the formats flagged for future removal ("BMP, HDF, PBM, PCX, PGM, and PPM files will not be supported in a future release" — noted in the Limitations/Version-History section of `print.html`).
- Background/canvas control: "By default, the canvas (background) of the exported model matches the color of the model. To use a white or transparent canvas for model files that you export to another file format, set the [Simulink Preferences] > [Diagrams/canvas] preference." (same `print-from-the-matlab-command-line.html` page)
- Resolution/DPI for Simulink diagrams specifically (whether `-r<dpi>` applies the same as for figures) is **not explicitly confirmed in the fetched Simulink-specific pages** — flagged as a gap below.

## 3. Layout/paper fitting, scale, orientation, tiling for large diagrams

**Source:** `print-from-the-matlab-command-line.html`, "Model Parameters for Printing" table (set via `set_param`):

| Parameter | Description/Values |
|---|---|
| `PaperOrientation` | `'portrait' \| {'landscape'}` |
| `PaperPositionMode` | `auto` – diagram is sized to fit the printed page (or to on-screen size for image export); `tiled` – print/export spans multiple pages. |
| `PaperType` | `'usletter' \| 'uslegal' \| 'a0'…'a5' \| 'b0'…'b5' \| 'arch-A'…'arch-E' \| 'A'…'E' \| 'tabloid'` |
| `PaperUnits` | `'normalized' \| {'inches'} \| 'centimeters' \| 'points'` |
| `TiledPaperMargins` | `[left top right bottom]` margin vector, default 0.5 in each |

**Default behavior for large diagrams (documented explicitly):** "By default, each block diagram is scaled during the printing process so that the diagram fits on a single page. In the case of a large diagram, this automatic scaling can compromise the printed image." — i.e., the default is scale-down-to-fit on one page/image, **not** cut-off and **not** auto-tiling.

**Tiled/multi-page option (opt-in, not default):**
```matlab
set_param('sldemo_enginewc/Compression', 'PaperPositionMode', 'tiled');
print('-ssldemo_enginewc/Compression', '-tileall')
```
Page-range selection: `print('-svdp','-tileall','-pages[2 4]')` (row-major page numbering). Page boundaries can be shown with `set_param(model,'ShowPageBoundaries','on')`. Note: "If you enable the print frame option, then Simulink does not use tiled printing."

## 4. Size/complexity limits, known limitations, caveats

- **Headless/no-display is explicitly unsupported — the key constraint for an autonomous agent.** From the `print` function reference's Limitations section:
  > "Starting MATLAB in no display mode on Linux or using the `-noFigureWindows` startup option on any platform has these limitations for `print`: Printing or saving figures with visible uicontrols errors. Always uses the painters renderer, even if you specify the `'-image'` option. In MATLAB Online™, `print` only prints to PDF... **When MATLAB is in no display mode, all Simulink printing functionality, including printing to a file, is turned off.**"

  This means the common headless/CI invocation `matlab -nodisplay` **cannot** be used for this workflow. A real display/X server equivalent (e.g., Xvfb on Linux providing a virtual display so MATLAB is not in "no display mode") is required for `print -s...` to work for Simulink diagrams at all.
- **Only one diagram level per `print` call** — loop `print` once per subsystem for multi-level export.
- **EPS format is unsupported for Simulink diagrams** (explicit note on `print.html`).
- **Subsystems/masks are not expanded by default**, per [Print Model Diagrams](https://www.mathworks.com/help/simulink/ug/print-model-diagrams.html):
  > "By default, Simulink does not print masked subsystems or subsystems linked to libraries. To print the contents of masked subsystems, in the Print Model dialog box, click **Look under mask dialog**. To print the contents of library links, in the Print Model dialog box, click **Expand unique library links**... You cannot print the contents of protected models."
  > "To print a model reference hierarchy, you must print each referenced model separately."
- **Print frames disable tiling** (see §3) — relevant if combining annotated output with tiling.
- The model must be loaded/open in the MATLAB session (not merely a file on disk).
- **No explicit numeric size/complexity ceiling is documented** (e.g., no stated max block count); the only documented consequence of a large diagram is automatic down-scaling to fit one page/image unless tiling is explicitly enabled.

## 5. Safe temporary-file handling

- `tempname` / `tempdir` are the documented primitives for avoiding littering the model's folder — [`tempname` function reference](https://www.mathworks.com/help/matlab/ref/tempname.html):
  > "`tmpName = tempname` returns a character vector containing a unique path for a temporary file in your system's temporary folder. `tmpName = tempname(folderName)` returns a unique path for a temporary file in the specified folder."
  Documented example:
  ```matlab
  filename = tempname;
  fileID = fopen(filename,'w');
  fwrite(fileID, magic(5));
  fclose(fileID);
  ```
  Documented **limitation**: "In most cases, `tempname` generates a universally unique identifier (UUID). However, if you run MATLAB® without JVM™ software, then `tempname` generates a random character vector using the CPU counter and time, which is not guaranteed to be unique." — relevant if an agent runs MATLAB with `-nojvm` for performance, since it could risk name collisions.
- There is no Simulink-print-specific temp-dir guidance beyond general use of `tempname`/`tempdir` as the `filename` argument to `print`, e.g.:
  ```matlab
  outFile = fullfile(tempdir, [tempname '.png']); % or simply tempname with a .png suffix
  print('-smodel', '-dpng', outFile);
  ```
  This combined pattern follows directly from the two documented APIs but is not shown verbatim as a single recipe in the official docs — flagged as an inference below.
- Simulink Report Generator's `Diagram` reporter documents its own temp handling: "The snapshot image is stored in the temporary folder of the report. When the report is closed, the snapshot image is copied into the report and then, the image is deleted from the temporary folder. To prevent the snapshot image file from being deleted, use the `Debug` property of the report." ([`slreportgen.report.Diagram` class reference](https://www.mathworks.com/help/rptgenext/ug/slreportgen.report.diagram-class.html))

## 6. Alternatives for an on-screen snapshot

- **`getframe`** ([function reference](https://www.mathworks.com/help/matlab/ref/getframe.html)) is a generic MATLAB figure function for capturing a *figure* frame as raster `cdata`; it is **not documented as a Simulink-diagram-specific capture mechanism**, and MathWorks notes it is being scoped down: "In a future release, the `getframe` function will no longer support capturing the figure toolbar, menu bar, or the border around the figure using the `rect` argument... As an alternative, you can use the `exportapp` function..." Not the recommended path for Simulink diagrams.
- **`print(..., '-RGBImage')`** is documented in `print.html` to return in-memory RGB `cdata` instead of writing a file — but it explicitly **excludes Simulink**: "You can also specify the resolution, renderer, and fig options with this syntax. **However, you cannot specify a Simulink block diagram.**" So the in-memory route is unavailable for Simulink diagrams; a file-based `print -s...` call is required, then the file can optionally be read back (e.g., with `imread`) for an in-memory array.
- **Simulink Report Generator** (`slreportgen.report.Diagram`, namespace `slreportgen.report`) is a documented, higher-level alternative for snapshotting a diagram into a report:
  > "Create a diagram reporter, including a diagram snapshot and caption, for a Simulink® or Stateflow® diagram... `diagram = Diagram(source)` creates a diagram reporter for the Simulink or Stateflow diagram specified by source. Adding this reporter to a report creates a snapshot of the diagram... displays in the report as an image with a caption."
  - `Source` accepts: the name of an open/loaded Simulink model, a subsystem path, a block handle, or a `Stateflow.Chart`/subchart object.
  - `SnapshotFormat` property controls the output image format; version history notes "Default value of SnapshotFormat is 'svg' for all report types" and "Removal of BMP for image output" — SVG is now the default snapshot format, BMP was removed.
  - **Licensing requirement:** this class lives under the `rptgenext` (Report Generator) documentation namespace and requires a **Simulink Report Generator license** — a separate, add-on product from base MATLAB/Simulink.
  - Heavier-weight than plain `print -s`, more suited to generating full documents/reports (PDF/Word/HTML) than quick single-image QA snapshots, and still subject to the same display-mode dependency as `print` (underlying diagram rendering requires a display).

---

## Gaps / uncertainties

- Direct access to `www.mathworks.com` was blocked (403 "Access Denied") from the research sandbox; all citations above were retrieved via the Wayback Machine mirror of the same official MathWorks URLs (raw HTML fetched and manually stripped to extract exact text), cross-checked against an independent AI web-search pass over the same pages, which agreed. Canonical URLs are given above; Wayback snapshot timestamps used during this research: `print-from-the-matlab-command-line.html` → `20241006045843`; `print.html` (MATLAB function reference) → `20250219063334`; `print-model-diagrams.html` → `20241205011946`; `tempname.html` → `20190822182720`; `getframe.html` → `20241209181547`.
- No official MathWorks page has a dedicated heading like "Limitations of Printing Simulink Models" — the headless-mode limitation is documented as a bullet within the generic `print` function reference page's Limitations section (not a Simulink-specific page), but it explicitly names Simulink and is treated here as authoritative.
- Resolution/DPI control for Simulink `print -s -dpng` specifically (whether `-r<dpi>` applies to Simulink diagrams the same way it does to figures) is **not explicitly confirmed or denied** in the fetched Simulink-specific pages. The generic `print` resolution argument (`-r<dpi>`) is documented for figures, and nothing in the Simulink pages contradicts it applying equally, but no explicit Simulink-specific statement was found. Recommend a follow-up fetch of the "resolution" input-argument section of `print.html` for higher confidence before relying on DPI control in an implementation.
- No explicit documented numeric limits on model size/block count for `print` were found — only the default-scaling-vs-tiled behavior (§3/§4).
- The `tempdir`/`tempname` + `print -s` combined recipe in §5 is a reasonable inference from two separately documented APIs, not a verbatim documented recipe for Simulink printing specifically.
