# BITS Pilani Grade Moderator

Browser-only JavaScript tool (no build step, no backend) used at BITS Pilani for grade moderation. An instructor loads marks and an ERP CSV, sees a marks-distribution histogram, moves grade cutoff lines to the right places, then exports the ERP CSV with grades filled in. Maintainer contact (shown in footer): Aritra Mukherjee, Dept. of CSIS, Hyderabad Campus.

## Repository

- GitHub: https://github.com/Aritra/GradeModerator.git (owner `Aritra`, repo `GradeModerator`), default branch `main`.
- **The GitHub `main` branch is the source of truth (latest stable version).** Pull before starting work (`git pull`); push only after the user approves the changes.
- Make small, focused commits with clear messages (one feature or fix per commit). Do not force-push, and do not push without the user's go-ahead.
- No build output: the files in the repo are the app as-is (open `index.html` directly in a browser).

## Working preferences (important)

- **Do not change coding logic** unless asked. Requests are surgical: refactor, fix bugs, add one feature. Keep edits minimal and targeted; do not restructure or "improve" unrelated code.
- Comment non-obvious logic. Prefer extracting shared helpers over duplicating code.
- Plain HTML + inline `<script>`; no bundler, no framework. Keep it that way.
- **Every time a feature is added or changed, update `help.html` (the searchable help page) and `README.md`** where they describe it, in the same piece of work.

## Files

| File | Role |
|---|---|
| `index.html` | Landing page: three equal buttons with 1rem gaps, "New Grade" (blue, `new_grade_updated.html`), "View Grades" (green, `view_grade.html`), "Help" (orange `.btn-dark-orange`, opens `help.html` in a new tab). The `.home-buttons` row sits in the same `col-sm-10` as the art with margins 6.45% / 7.65% = the art SVG's blank edges, so it spans the visible artwork end to end. Footer links to YouTube tutorial, `help.html`, maintainer email. |
| `new_grade_updated.html` | The only new-grade page ("New Grade" on the home page): paste two tab-separated columns (ID, Marks) from Excel/Sheets into a textarea, pick start row, upload ERP CSV, build the grade object, show special-grades modal, go to histogram. Fixed columns (`chosenIDIdx = 0`, `chosenMarksIdx = 1`), status messages in `#demo`, `escapeHtml()` in the modal. The older CSV-upload page `new_grade.html` was deleted (recoverable from git history). |
| `view_grade.html` | Upload a previously saved grades JSON, store as `gradeObject`, go to histogram. |
| `histogram.html` | Main app: histogram, cutoff editing (draggable grade bubbles), grade analytics, student search, export (ERP CSV, JSON, print). |
| `histogram_tables.html` | Fallback: copy of `histogram.html` just before the Search/Edit Grades "change to non-letter grade" feature (fluid layout, grade tables, hover popups, analytics fixes all included). Not linked; reach it like the other fallbacks. Do not edit it. |
| `histogram_bubbles.html` | Fallback: copy of `histogram.html` with draggable bubbles but before the fluid-sizing change (fixed px fonts, %-height rows). Not linked; reach it the same way as `histogram_classic.html`. Do not edit it. |
| `histogram_classic.html` | Fallback: unmodified copy of the GitHub `histogram.html` from before the bubble redesign (radio buttons + slider editing). Not linked from any page; reachable by changing the URL from `histogram.html` to `histogram_classic.html` in the same tab (same `sessionStorage`). Do not edit it. |
| `sortStudents.html` | Opened (new tab) from histogram's "Save ERP CSV" → "Sort by Branch/Campus": sort ERP rows by campus / affiliation / both, then export CSV. |
| `help.html` | Searchable user guide: 18 `.help-section`s on how the system works (input, matching, histogram, cutoffs, MGPV, non-letter grades, Search/Edit, exports, refresh, ID format, troubleshooting), auto-built table of contents, sticky search box that shows only sections containing all typed words and highlights them (`<mark>`; each search restarts from the saved section HTML). Keep it in sync when behaviour changes. |
| `print.css` | Print stylesheet linked by `histogram.html`: hides `#keepGradeDiv`, `#changeGradeDiv`, `#allControlsBelow`. |
| `imgs/` | `moderator_art_2.svg` (landing art), `separator.svg`, plus unused logos/icons/fonts. |
| `README.md`, `LICENSE.md` | User-facing readme (MIT licence). |

Libraries via CDN: Bootstrap 5.3.3 (all pages), Chart.js **2.9.4** (histogram only, v2 API, do not upgrade casually), jQuery 3.7.1 slim (histogram only, effectively unused).

## Data flow

```
new_grade_updated.html ─┬─ sessionStorage["gradeObject"] ──> histogram.html
view_grade.html ────────┘                                        │
                                                                 ├─ Save Grades -> downloads JSON (reloadable via view_grade.html)
                                                                 ├─ Save ERP CSV -> "Export Directly" -> preview tab + CSV download
                                                                 └─ Save ERP CSV -> "Sort by Branch/Campus"
                                                                      sessionStorage["gradeObject2"] = {retCSV, retCSVName}
                                                                      └─> sortStudents.html (new tab)
```

histogram.html keeps `sessionStorage["gradeObject"]` up to date while you work (`buildGradeObject()` + `saveSessionState()`: debounced 300 ms at the end of `updateAfterCutoffChange()`, and on `pagehide`), so a browser refresh restores the current cutoffs, kept grades and non-letter changes instead of the state the page was opened with. `exportJSON()` → `downloadGradeJSON('')` uses the same `buildGradeObject()`. On load, a saved `histRange` overrides the range computed from `mMap` (otherwise moving the lowest/highest student to a non-letter grade would narrow the range on reload and could leave a cutoff outside it); JSON files without `histRange` behave as before.

Subject code is derived from the ERP CSV filename: `parts = name.split("_")`, `code = parts[1] + " " + parts[2]`.

### Grade object (`gradeObject`)

```
{ code, subjectName, fullMarks,
  mMap:        [[studentID, marks, name], ...]            // letter-graded students
  specialGMap: [[studentID, specialGrade, name, marks], ...]
  retCSV:      2D array, the ERP CSV (row 0 = headers)
  specialG:    [0,0,0,0,0,0,0,0]                          // legacy placeholder
  keepG:       boolean[8]                                 // which letter grades are active
  gCutOffs:    int[8]                                     // min marks per grade, [] for a new session
  retCSVName,
  histRange:   [markLowest, markHighest]                  // optional; written by histogram.html (autosave + Save Grades)
}
```

### ERP CSV column constants (must stay in sync across files)

`ERP_ID_COL = 3`, `ERP_NAME_COL = 4`, `ERP_MARKS_COL = 6`, `ERP_GRADE_COL = 7`. Defined separately in `new_grade_updated.html`, `histogram.html`, `sortStudents.html` (only the ones each file needs).

## Domain rules

- **Letter grades** (index 0..7): `E, D, C-, C, B-, B, A-, A`, grade points `2, 4, 5, 6, 7, 8, 9, 10`. MGPV = weighted mean grade point over active grades.
- **Non-letter (special) grades**: `W, I, NC, GA, RC, DP, TGA, RRA` (`SPECIAL_GRADES` set in `new_grade_updated.html`). Students whose ERP grade is one of these are kept out of the histogram and listed separately. The ERP CSV is the master list. A student in it but missing from the pasted marks: if their ERP grade is W → `specialGMap` with marks 0; otherwise they are collected in `missingStudents` and `showMissingMarksModal()` asks for their marks (all boxes required, 0..max, decimals `Math.ceil`; Cancel to fix the paste) before continuing, then they are placed like pasted students (non-letter ERP grade → `specialGMap` with the marks, else `mMap`). Pasted students not in the ERP CSV are ignored.
- **Histogram range**: min/max marks across `mMap`, extended by 5 on each side, clamped to `[0, fullMarks]`. New sessions (`gCutOffs == []`) space the 8 cutoffs evenly (`step = floor(range/8)`).
- A grade is assigned by scanning from A down to E: first active grade where `marks >= cutoff` (`getGradeFromMarks`).
- Per AUGSD, the "Keep Grades" checkbox row stays hidden. (Before the bubble redesign the page also showed the "Change Grades" radio row and a slider behind an "Enable editing" switch; see `histogram_classic.html`.)

### Student ID format (13 characters, always)

`[0-3]` admission year, `[4-7]` affiliation code, `[8-11]` roll number, `[12]` campus letter (`H` Hyderabad, `P` Pilani, `G` Goa).

Affiliation (4 chars): single degree is `<branch2>PS` (e.g. `A7PS`), dual degree is `<branch1><branch2>` (e.g. `A7B4`). Branch code to programme name lookup is the `BRANCH_CODES` map in `sortStudents.html` (A*, B*, C* first-degree/MSc codes and 2-digit higher-degree codes).

An earlier wrong format (leading f/h/p, 14 chars) has been corrected; do not reintroduce it.

### Student ID matching

IDs are matched **everywhere** on year of admission + roll number + campus letter only, ignoring the programme code (chars 5–8), case and surrounding spaces: `idMatchKey(id)` = chars 1–4 + chars 9–13 of the trimmed, upper-cased ID (a non-13-char ID is compared whole). The same helper is defined in `new_grade_updated.html` (marks ↔ ERP matching in `openHistogram()`) and `histogram.html` (`indexFromMarkMap` / `indexFromSPGMap`, used by the ERP export `fillGradesIntoCSV()` and Search/Edit Grades); keep the two copies identical. Students are stored with the ERP's form of the ID. The Search/Edit text filter (`spgSearch`) is a plain substring search, not ID matching.

## new_grade_updated.html

- `paste` listener on `#pasteMarksData` calls `handlePaste()`: split on `\r?\n` and `\t`, strip leading/trailing empty rows and trailing blank cells, sanity-check exactly two columns (rows with more are shown red and the paste is rejected; one column only is rejected), pad rows to 2 columns, round decimal marks in column 2 up (`Math.ceil`; non-numeric cells untouched), write the cleaned data back into the textarea, store in `csvSRCtxt`. `renderPastePreview()` shows the rows in `#pastePreview` (first cell = 1-based row number matching the start-row dropdown); `highlightPastePreview()` colours rows before the chosen start row as headers (dark blue `#0a1f66`, bold white) and the rest as data (light yellow). Clearing the textarea resets state. Extra validation in `openHistogram()` (pasted data, start row, ERP loaded).
- The ERP upload has a hint and a bold red "do not rename the ERP file" warning (subject code comes from the filename), and the submit handler rejects files not ending in `.csv`.
- `openHistogram()` loops over ERP rows (from row 1), linear-searches the marks rows for an ID match via `idMatchKey()` (see *Student ID matching*), stores the student under the **ERP's** ID, splits into `markMap` / `specialGradeMap`, asks for missing marks if needed (`showMissingMarksModal`, continues on `hidden.bs.modal` so modals don't overlap), then `proceed()` shows the special-grades modal if needed (OK → store + navigate), else navigates directly.
- `csvToArray` is a plain `split("\n")` / `split(",")`.

## histogram.html: how it works

- **Loading overlay** (`#pageLoader`, first element in `<body>`, pure CSS spinner): visible from first paint; `loadAtStart()` ends with `hidePageLoader()` (fade after two animation frames, then removed). `showLoadError(msg)` (inline script right after the overlay, before the libraries) replaces the spinner with a message + home link: used when `gradesDetails` is null (page opened without session data) and by a capture-phase `window` error listener (script errors or a failed library `<script>` download while the overlay is up). **The Bootstrap JS, jQuery and Chart.js `<script>` tags are at the end of `<body>`, just before the page script** (not in `<head>`), so the overlay can paint while they download; they still run before the page script. Keep them there.
- Single `mainChart` (Chart.js bar). Created once in `redrawMainHistogram()`; afterwards only `config.lineAtIndex` is updated and `update('none')` called. **Never call `mainChart.destroy()`**: it resets canvas sizing and made the canvas overflow the screen (commit `813258c`, "Fixed histogram glitch on edit").
- `verticalLinePlugin` (registered globally via `Chart.plugins.register`) draws each cutoff line at `getCutoffX(chart, idx)` (left edge of the bar, from `meta.data[i]._model`, Chart.js v2 internals). Grade names are no longer drawn on the canvas; the bubbles show them. After drawing, if the chart is the main chart (`chart.canvas === myCanvas`), it calls `positionGradeBubbles(chart)`, so bubbles follow every draw (first render, cutoff change, window resize). Because the plugin is global, it also runs on the analytics chart (harmless: no `lineAtIndex`, different canvas).
- `updateAfterCutoffChange()` is the single path for any cutoff or keep-grade change: recompute populations, refresh the cutoff/count cells, MGPV, graded count, redraw chart.
- **Cutoff editing = draggable grade bubbles** (the "Change Grades" radio row, the slider and the "Enable editing" switch were removed; a spacer column keeps the bottom buttons in place). One "ball on a stick" per grade (`gradeBubbles`, created once in `#gradeBubbleArea`, only moved/shown/hidden afterwards so a drag keeps its pointer capture while the chart redraws). Ball and stick are `#000088`, same as the line. Consecutive active grades alternate stick length (`BUBBLE_STICK_REM = [0.375, 2.25]`, ball `BUBBLE_BALL_REM = 1.75`, must match the CSS) so neighbouring bubbles don't overlap; chart `layout.padding.top` = `bubbleHeadroomPx()` reserves room for the tallest one.
  - Positioning is in **% of `#gradeBubbleArea`**, which is laid out exactly like the canvas (absolute 100% x 100% child of `.row.wrapper`, same Bootstrap `.row > *` side padding) and so covers the box the chart is drawn into. Chart.js lays out in `chart.width` px but the browser draws that squeezed into the canvas content box (Bootstrap padding + `box-sizing: border-box`), and the canvas is re-stretched when printing; % follows both. Pixel offsets did not (bubbles drifted right and were wrong in print).
  - `startBubbleDrag(evt, g)` uses pointer events (mouse, touch, pen; `touch-action: none`). The pointer is converted to chart px via `#gradeBubbleArea`'s rect, snapped to the nearest bar edge, and clamped strictly between neighbouring active cutoffs (`findImmediateLeft/Right`; lowest grade can go down to `markLowest`, highest up to `markHighest`), the same limits the old slider had.
- Keep/remove grade checkboxes (`#keepGradeDiv`, always hidden per AUGSD) go through `reDrawOnGradeAlter(g)` (only after `initialLoadingDone`); `findImmediateLeft/Right` find neighbouring active grades (right sentinel is 8). Re-adding needs a gap ≥ 2; at least one grade must remain. Bubbles of inactive grades are hidden.
- Subject title (`#subjectTitle`): `#TFSubjectCode` + `#TFSubjectNameText`, with a pencil/tick button (`#titleEditBtn`, inline SVG icons, hidden in print) on the left. `toggleTitleEdit()` hides the title text and shows `#subjectCodeInput` (8em) + `#subjectNameInput` (rest of the row); tick or Enter in either box → `freezeTitle()` (an empty box keeps its old value), Esc cancels. A new code updates `subjectCode`/`gradesDetails.code`, the browser tab title and the Save Grades file name; the ERP CSV export keeps the original ERP file name (`retCSVName`). A new name goes to `gradesDetails.subjectName`. Both are in `buildGradeObject()`, so they are in the saved JSON and survive a refresh.
- Bar click (`myCanvas.onclick`, chart `events: ['click']`) opens `#myModal` listing students at that mark and their current grade.
- Other helpers: `recalculateGradePopulation`, `updateGradeDisplay`, `calculateMGPV`, `fillGradesIntoCSV` (writes grade + marks into `retCSV`; unknown ERP IDs get `W`/0 and an alert), `arrayToCSV`, `indexFromMarkMap`, `indexFromSPGMap`.
- Search/Edit Grades overlay (`#overlaySG`, flex column `.sg-layout`): `drawTable()` clears then fills `#specialGradeTable` with letter-graded students (clickable rows, `tr.sg-editable`, `data-id`) plus special-grade students (read-only, excluding W); `spgSearch()` filters by ID or name.
- **Changing letter grades to non-letter grades** (Search/Edit Grades): click a letter-graded row → `selectSearchStudent()` fills `#sgEditBar` (`ID · Name · marks`) and `#sgGradeSelect` (first option = current letter grade, value `""`; then I, NC, GA, RC, DP, TGA, RRA via `awardableNonLetterGrades()`: W is never offered, it only comes from the ERP). `onSearchGradeChange()` records/cancels the change in `pendingSpecial` (Map id → code); pending rows show `B- → NC` in red (`tr.sg-pending`). Several students can be pending at once. A bold red `#sgWarning` above "I am done!" explains there is no undo (restart from the files/JSON; reverting after cutoff changes would silently re-band students and change the moderated counts/MGPV). `finishSearchGrades()`: no pending → just closes; otherwise `confirm()` listing the changes (first 20), Cancel keeps the overlay open, OK → first `downloadGradeJSON('AUTO_BACKUP_')` (pre-change state, same name as Save Grades with the prefix; an underlined `#sgBackupNote` in the overlay explains it), then `applyNonLetterGrades()`: moves each student from `markMap` to `spGMap` as `[id, code, name, marks]` (marks kept, so Save Grades JSON and ERP export carry them), removes them from `yvalues` / `marksForHistogram` / `marksDistribution` in place, recomputes Average Marks, `updateNonLetterCounts()` (non-letter cells + Total Students, shared with `loadAtStart()`), then `updateAfterCutoffChange()` (grade counts, MGPV, Students Graded, chart). **`gCutOffs` and the x range (`markLowest..markHighest`) are deliberately not changed**, so lines and bubbles stay put even if the removed student had the lowest/highest mark. A change that leaves no letter-graded students is allowed (MGPV 0).
- Grade Analytics overlay (`#overlayGA`, nested inside `#overlaySG`'s DOM): `renderAnalyticsChart(type, title, legendPos)` for bar/pie over active grades only, tracked in `analyticsChart` and destroyed before redraw; `toggleChart()` switches; `isHistogram` is true while the bar chart shows, and `overlayGA()` resets it (and the button label) because the overlay always opens on the bar chart. Options use Chart.js 2.9 names (`title`, `legend`, `tooltips`, `scales.yAxes` with `precision: 0`).
- Hover popups on the grade tables (one shared `#gradeTip`, wired by `attachGradeTip(table, htmlFor)`, built on each hover so it matches the current state; positioned near the pointer and kept inside the window by `placeGradeTip()`; hidden in print; mouse only, no touch):
  - `#letterTable` → `mgpvTipHtml(cellIndex)`: a grade column shows its weight (grade points), student count, weight × students and share of the MGPV; the label column shows all weights and the full formula.
  - `#nonLetterTable` → `nonLetterTipHtml(cellIndex)`: a grade column lists ID and name of the students in `spGMap` with that grade (first `TIP_MAX_STUDENTS` = 25, then "… and N more"; text escaped with `escapeHtml`); the label column shows the count per grade. `NON_LETTER_GRADES` (W, I, NC, GA, RC, DP, TGA, RRA, table column order) is shared with `loadAtStart()`'s count tally.
- Special-grade count cell for DP has id `spGradeTP` (`tfCountTP`).

### Fluid sizing (scales with the screen)

- `html { font-size: clamp(12px, calc(6px + 0.35vw + 0.35vh), 24px) }`: 1rem is ~12.7px at a 1280x633 viewport, ~16px at 1920x993, ~19.7px at 2560x1353. All of this page's sizes are in rem (Bootstrap already is), so text, buttons, bubbles, overlays (`max-width: 95vw; max-height: 95vh`) and the search table scale together. Keep new sizes in rem, not px.
- Layout is a CSS grid on `#mainLayout` (the main `.container-fluid`): `auto` top panel, chart `minmax(0, 1fr)` (top fixed under the top panel, fills down), buttons area `10vh`: `#allControlsBelow { padding: 2vh 0 3vh }` with `.btn { height: 5vh; font-weight: bold }`, i.e. 2% gap below the chart's x-axis labels, 5% buttons, 3% gap to the window bottom. In print the last track is `auto` (buttons hidden, chart fills the page). Don't reintroduce % heights inside Bootstrap rows: a stretched flex column is a definite height, so % children become fixed slices and squash their content. `#keepGradeDiv` is hidden with `display: none` (not `visibility`) so it takes no space.
- Top panel, left: `#statsRow` (Total Students, Students Graded, Maximum Marks, Average Marks, MGPV) uses larger sizes (labels 1.2rem, numbers 2.25rem, "MGPV" 1.6rem, `#TFMGPV` 5.25rem with `line-height: 1`); each column is a flex column so numbers line up when labels wrap.
- Top panel, right (`#gradePanel`, flex column, top-aligned): two bordered tables `.grade-table` (1px solid black, `table-layout: fixed`, font 1.1rem; label column `10.75em` = bold "Non Letter Grades" ~9.15em + ~0.6em padding each side, `nowrap`, same in both tables so the grade columns line up; the 8 grade columns share the rest): `#nonLetterTable` (header row `tr.head-grey`: #555555, bold white; then Count with `spGrade*` ids) and `#letterTable` (header row `tr.head-navy`: #000080, bold white; then CutOff `cutOff*` and Count `count*` ids). The coloured `separator.svg` strip (`.grade-strip`, full table width, no margins) sits flush between the two tables; it's a 100:1 SVG, so its height follows its width (~9px at 1920 wide). The top panel row has `padding-bottom: .75rem`, which keeps a gap between the tables and the chart (and highest bubble). The `<td>` ids are the same ones the script always wrote to (`tfCutOffs`, `tfCounts`, `tfCount*`).
- Chart.js text: `applyChartFontSize()` sets `Chart.defaults.global.defaultFontSize = round(0.75 rem)` before creating either chart; the main chart's `onResize` re-applies it and recomputes the bubble headroom (v2 calls `onResize` before re-layout; tick fonts resolve from the global default at draw time).
- `.btn-xs` line-height changed from `.5` (squashed) to `1.2`.
- Main histogram y-axis: `ticks: { beginAtZero: true, precision: 0 }` so small classes show whole-number counts (0, 1, 2), not 0.2 steps.
- Verified in headless Chrome at 1280x720, 1366x768, 1920x1080, 2560x1440 (no overflow, bubbles within 0.5px of lines) and in print-to-PDF.

### Print PDF flow

1. "Print PDF" calls `showNCDialog()`: Bootstrap modal `#ncDialog` with a 200-char textarea (`maxlength=200`) and live counter (`updateNCCounter()`).
2. "Print" calls `confirmAndPrint()`: writes the text into `#ncCutoffPrintText`, hides `#allControlsBelow`, shows `#divNCcutoff` (same grid slot, `height:10%`, with a ✕ button calling `hideNCCutoffDiv()` to restore the controls). Printing starts from the modal's `hidden.bs.modal` event, after removing any leftover `.modal-backdrop`, then `printpage()` (injects `@page { size: landscape; }`).
3. Inline `@media print` hides `.overlayDiv`, `.modal-backdrop`, `#myModal`, `#ncDialog`; `print.css` hides the keep-grade row and the controls bar (its `#changeGradeDiv` rule is now a no-op). Grade bubbles print (`print-color-adjust: exact`) and stay on their lines (verified via headless Chrome print-to-PDF).

### CSS gotchas

- The bar-click student-list modal CSS is scoped to `#myModal ...`. Using bare `.modal`, `.modal-content`, etc. overrides Bootstrap's own modal and put `#ncDialog` under the `.overlayDiv` backdrop (z-index 1000). Keep it scoped.
- `.overlayDiv` is a full-screen translucent backdrop toggled via `visibility`. It must not be visible when printing.
- The histogram canvas is a direct child of a Bootstrap `.row`, so it gets side padding and its drawing is squeezed relative to Chart.js's own coordinates. Anything overlaid on the chart must map coordinates through the canvas content box (see the bubble positioning), not raw `_model.x` pixels.

## Testing

No test suite. Changes have been checked with headless Chrome (`google-chrome --headless=new`): copy a page to the scratchpad, inject a `<script>` that seeds `sessionStorage["gradeObject"]` and/or simulates events (`ClipboardEvent('paste')` with a `DataTransfer`, `PointerEvent`s with `Element.prototype.setPointerCapture` stubbed, `formDES` submit with a `File`), write results into a `data-out` attribute, read them back with `--dump-dom`; use `--screenshot` and `--print-to-pdf` + `pdftoppm` for visuals.

## sortStudents.html

`erpHeaders` comes from `retCSV[0]` (blank headers filtered out); rows are converted to objects keyed by header with derived `_campus`, `_affiliationRaw` (sort key), `_affiliationDisplay`. Sort options (radios): original / campus / branch (affiliation code) / both (campus then affiliation); always re-sorted from the untouched `enrichedData`. Export writes only the original ERP columns, every value double-quoted, in the currently displayed order (`currentSortedData`), via Blob download. Campus and Branch columns are display-only.

## Bugs already fixed (don't regress)

- `== NaN` check on max marks (now `isNaN`).
- Implicit globals in `spgSearch()` and `loadAtStart()`.
- `findImmediateRight` out-of-bounds read (`i <= 8` -> `i < 8`).
- Duplicate rows each time the Search/Edit Grades table opened (rows now cleared first).
- Stacked Chart.js instances / canvas overflow on edit (now a single `mainChart`, never destroyed).
- sortStudents: wrong ID offsets, header rendering via `Object.keys` on an array, export ignoring sort order.
- Bare `.modal` CSS overriding Bootstrap modals (now scoped to `#myModal`).

## Known limitations / things to verify (observed in code, not yet fixed)

- `mainChart.update('none')`: in Chart.js 2.9.4 a non-object argument is treated as a duration; it works only because `animation: false` is set.
- `overlayGA()` "average lies in grade" label loops `i < 7` and ignores `keepG`: an average in the A band (or near disabled grades) gives an empty/wrong label.
- `arrayToCSV` (direct ERP export) drops blank cells, so an empty cell in the middle of an ERP row shifts later columns left. `sortStudents.html` export does not have this problem.
- `fillGradesIntoCSV` leaves the old ERP grade in place if a student's marks are below every active cutoff.
- `csvToArray` is a plain `split(",")`: quoted fields containing commas are not supported; `\r` from CRLF files stays on the last cell of each row (trimmed only where `.trim()` is called).
- In `sortStudents.html`, blank header cells are filtered out of `erpHeaders` but row values are read by the filtered index; a blank header in the middle of the row would misalign columns (trailing blanks are fine).
- `getCampusFromID` / `getAffiliationFromID` return "Unknown" for any ID that is not exactly 13 characters after trimming.
- Tutorial link differs: `index.html` uses `https://youtu.be/-dX6KwiFTj4`, `README.md` uses `https://youtu.be/ez0jZY4h_zU` (README was updated most recently).
