# 0001 — Mobile support

- Status: In progress — upstream mobile foundation integrated; fork acceptance remains incomplete.
- Created: 2026-09-09
- Last updated: 2026-09-09
- Constraints: [AGENTS.md](../AGENTS.md), [DESIGN.md](../DESIGN.md)
- Related evidence: [UI transitions](../verification/UI-TRANSITIONS.md), [looping archive](../verification/LOOPING-ARCHIVE.md), [model viewer](../verification/MODEL-VIEWER.md), [render quality](../verification/RENDER-QUALITY.md), [data repository](../verification/DATA-REPOSITORY.md)

## Background

At the initial planning baseline, the interactive archive targeted a desktop display. Phones can reach the generated reading catalogue and articles, but cannot use the full archive experience. Mobile support should make browsing, reading, search, saved archives, settings, and model inspection usable with touch in portrait and landscape.

### Initial implementation (before upstream integration)

- `src/entry.ts` starts the interactive UI only when the viewport is at least 1000 × 620 CSS pixels and WebGL 2 is available. Smaller screens retain the catalogue. Crossing below that threshold after startup redirects to `/?view=list`.
- `src/main.ts` scales a fixed 1920 × 1080 stage to fit the window. `src/style.css` places navigation, archive details, dialogs, and viewer controls within that stage. Simply removing the entry gate would shrink text and controls excessively on phones.
- Archive navigation has buttons and keyboard actions, plus canvas pointer selection and limited detail rotation in `src/scene.ts`. It has no deliberate swipe navigation or multi-touch arbitration. Some feedback depends on hover.
- The independent viewer uses `OrbitControls` and smooth camera following, but its layout and instructions are desktop-oriented. Existing library behavior must be tested before defining any additional gesture handling.
- `src/quality-renderer.ts` also uses the desktop stage scale when sizing render buffers. Responsive canvas dimensions must be reconciled with that calculation.
- Generated catalogue and article pages already provide a lightweight reading path. They share content from `scripts/content.mjs`; mobile support must preserve their URLs, ordering, metadata, and content ownership.

### Scope and constraints

The proposed first release covers the interactive archive, boot and replay, detail reading, search, saved archives, settings, and the independent 360° viewer. Support starts at 320 CSS pixels wide, with representative phone and tablet portrait and landscape layouts. Small desktop windows also receive the compact layout; touch-capable wide screens retain usable touch controls.

Keep a single TypeScript / Three.js / Vite application and shared selection, preference, modal, and content state. Do not create a second mobile app or copy personal content into UI source. `RHINELAB_CONTENT_DIR`, existing Markdown and `site.json` inputs, and the 288-position interactive / 160-position cinematic pools remain unchanged.

Desktop retains the established 1920 × 1080 composition and reference timeline. Compact layouts may adapt UI placement and camera framing, with the exact framing validated during implementation. Preserve the approved assets, baseline lighting and materials, wave behavior, vertical-only extraction, rotation clearance, return order, and decryption. Camera adaptation must not move the extracted card sideways or lower surrounding cards to make space.

This feature does not include a native app, installation/offline support, new models, a visual redesign, or automatic translation. Browser fullscreen is optional; all primary tasks must work without it. Rendering optimization is a separately measured concern, not an assumed consequence of responsive layout work.

### Decisions still to validate

The layout thresholds, compact scene framing, and gesture thresholds below are initial design decisions. Validate them with screenshots and device interaction before marking the spec implemented. Record any changes here and in the log. Exact physical test devices and OS/browser versions must be recorded during implementation; this draft does not claim device compatibility or measured performance.

## Landing plan

### Proposed user experience

| Surface | Compact behavior |
| --- | --- |
| Entry and fallback | Start the interactive experience on supported small screens once the compact experience is complete. Keep `/?view=list` as an explicit lightweight choice. Missing WebGL 2, initialization failure, or unrecoverable context loss must leave a usable reading route. Resizing or rotating must not redirect or reload. |
| Boot and replay | Retain the white-start sequence and original timing. Fit the decorative reference composition into the available area while keeping identity text and a real-size ENTER SYSTEM control readable and reachable. Replay, skip, and reduced motion remain available without a keyboard. |
| Archive | Compact brand and system actions at the top, scene in the central area, selected title/ID and ACCESS FILE plus navigation below. Keep text and controls at CSS-pixel sizes instead of scaling the entire UI. Long titles wrap or truncate with an accessible full title. Hover is optional. |
| Detail | In portrait, place a bounded model preview above a readable document area, with back, bookmark, and viewer actions reachable. In landscape, use side-by-side placement only when both remain usable; otherwise keep the compact stack with scrolling. Preserve tabs and scroll position on bookmark, viewer return, and viewport changes. |
| Search, saved, settings | Use a full-width sheet or full-screen dialog on compact screens, with a visible close action and an independently scrolling body. Keep the search field and results accessible when the software keyboard is open. Preserve exit transitions, focus restoration, and background input isolation. |
| Independent viewer | Use the available viewport with safe-area padding, a reachable close action, wrapping controls for surface, explode, assemble and reset, and touch instructions. Provide visible zoom and directional pan buttons as alternatives to multi-touch gestures. Preserve smooth reset and interruption behavior. |
| Catalogue and articles | Keep direct reading URLs and browser navigation. Verify responsive navigation, directory cards, table of contents, images and long content. Code blocks, wide tables and formulas scroll inside their own container instead of widening the page. |

Use the existing 1000 × 620 threshold initially to choose between desktop composition and compact reflow, rather than as an eligibility gate. Base this choice on layout dimensions, not user-agent strings. Use pointer/hover capabilities for hints and control sizing independently of layout. Handle short landscape screens explicitly, including wrapping navigation and scrollable dialog content.

Account for dynamic browser bars and safe-area insets. A software keyboard opening must resize/reposition the active dialog without rebuilding application state or switching layouts repeatedly. Keep browser text selection and zoom available in reading surfaces. Primary touch controls must provide at least a 44 × 44 CSS-pixel hit area; compact reading text starts at 16 CSS pixels. These are project acceptance targets, to be checked at actual rendered size.

### Touch and input contract

| Context | Input | Expected result |
| --- | --- | --- |
| Archive scene | Tap a card | Select that card using the existing selection rules; ACCESS FILE explicitly opens the selected archive. |
| Archive scene | Swipe left / right | Advance to next / previous category, matching the next / previous column buttons. Wrap in the same physical travel direction and restore per-column selection memory. |
| Archive scene | Swipe up / down | Advance to next / previous archive in that category, matching the next / previous archive buttons, including wraparound. |
| Archive controls | Previous/next category or archive buttons | Provide every navigation action without requiring swipes. |
| Detail model preview | One-finger drag | Use existing bounded inspection rotation only after extraction clearance; never select another archive or scroll the document from this region. |
| Reading or dialog content | One-finger scroll, text selection, links | Normal reading interaction; never dispatch archive navigation. |
| Independent viewer canvas | One-finger drag | Orbit the model. |
| Independent viewer canvas | Two-finger pinch / translation | Zoom / pan through the existing camera controller with smooth following and current limits. |

Recognize one archive step per completed swipe, locking to the dominant axis after movement is intentional. Start with a 24 CSS-pixel displacement and a 1.5:1 dominant-axis ratio; test diagonal and slow movements, then log any tuning. Swipes must not also produce a tap. A second pointer, `pointercancel`, lost capture, orientation change, or a newly opened modal cancels any incomplete archive gesture without an action. A new gesture can start immediately after cancellation. Avoid `movementX` as the only source of touch deltas; track pointer IDs and client coordinates.

Limit gesture capture and restrictive `touch-action` rules to the interactive canvas regions. Keep browser edge navigation available. Do not let canvas handlers intercept controls layered over the scene or content scrolling. Mouse, keyboard and touch all dispatch the same navigation actions; hybrid devices must not duplicate actions or require a reload when input type changes.

### Implementation sequence

Upstream `5abab02` supplies much of stages 1–4 and parts of stage 5. The sequence below remains the acceptance roadmap; the comparison section records current coverage and remaining work. Each implementation commit must update this spec and its change log with behavior changes and actual validation results.

1. **Establish viewport and input foundations.** Capture desktop reference states and add a shared viewport/layout description. Separate the fixed cinematic coordinate system from responsive UI and scene hosts. Extract shared navigation dispatch only as needed. Identify fixed projection coordinates in `src/scene.ts`, inspection overlays, document decryption, and render sizing. Keep small-screen entry on the catalogue until the complete compact path is usable.
2. **Implement archive and detail layout.** Add compact styles, safe-area handling, readable navigation, bounded scene/preview regions, and document scrolling in `src/main.ts` and `src/style.css`. Adapt camera aspect, projection anchors, inspection overlays, and render dimensions to their actual host rectangles. Preserve desktop trajectories and model transforms. Make boot/skip/replay usable in the same layout without changing the reference timecodes. Depends on stage 1.
3. **Implement touch navigation.** Add gesture recognition and cancellation to archive input, reuse existing loop/selection state, and adapt hints and hit targets. Preserve bounded detail inspection and mouse/keyboard input. Depends on the canvas/control boundaries from stage 2.
4. **Complete dialogs, reading, and viewer.** Reflow modal content, quality/audio controls, tabs, and viewer actions; handle software keyboard changes and focus. Verify or explicitly configure `OrbitControls` touch mapping without a competing gesture recognizer. Add visible viewer pan/zoom actions using the existing camera motion. Audit generated reader/catalogue styles and adjust only where needed. Depends on stages 1–3.
5. **Validate resilience and performance.** Exercise slow/failed assets, WebGL failure/context loss, background/resume, rotation, and repeated viewer use. Route unrecoverable failures to reading without a retry loop. Preserve audio preferences and start/resume sound through an actual user interaction; do not replay missed boot sounds. Keep existing quality defaults and saved user choices. Measure actual render buffers and device frame times; fix sizing/resource problems first. Any proposed new default or automatic quality policy needs an explicit spec revision and visual evidence. Depends on stages 2–4.
6. **Enable and release mobile entry.** Remove the size-based startup rejection and resize redirect only after the acceptance checks pass. Keep explicit list mode and failure fallback. Update `README.md`, `DESIGN.md` for implemented compact composition, and a new `verification/MOBILE-SUPPORT.md` with screenshots, device results, and limitations. Commit/sync to this fork's `origin/main` under the repository workflow. A data-site rollout pins the verified UI commit SHA in its own deployment workflow; no article changes are needed. Depends on stage 5.

Likely implementation files are `src/entry.ts`, `src/main.ts`, `src/style.css`, `src/scene.ts`, `src/inspection-overlay.ts`, `src/document-decryption.ts`, `src/model-viewer.ts`, `src/viewer-camera.ts`, `src/quality-renderer.ts`, and settings styles. Inspect generated reader styles before editing their source. New viewport/gesture modules are appropriate if they isolate testable behavior. No model asset regeneration is planned.

If rollout exposes a critical interaction failure, restore the previous verified UI commit (and the consumer's pinned SHA if deployed). The reading catalogue remains available throughout. Document the regression and correction in this spec; do not silently mark the feature verified.

### Upstream integration comparison — 2026-09-09

Source: upstream `5abab02` (`feat: adapt archive UI to mobile and add offline PWA`), rebased over fork baseline `42572a4`. This section supersedes the initial implementation description and records deviations from the proposed design without treating them as fully accepted.

| Plan area | Upstream implementation / fork integration | Remaining work against this spec |
| --- | --- | --- |
| Viewport foundations | Shared `viewport-layout.ts`, CSS-pixel compact controls, camera-only framing, actual canvas buffer scale. Portrait means aspect < 1.05; compact means portrait, width < 1100, or coarse pointer with height < 600. Desktop uses a 1080-unit logical height with variable width. | These replace the proposed 1000 × 620 layout threshold. Validate more tablet, hybrid-input and extreme short-window combinations. 1920 × 1080 remains the reference. |
| Mobile entry | Remove this fork's old size gate and resize redirect; preserve explicit `?view=list`, generated catalogue, and initialization fallback. | Complete acceptance is not a prerequisite for importing this usable upstream foundation under the user's rebase request. Runtime WebGL context-loss recovery remains unimplemented. |
| Archive and detail | Portrait preview above independently scrolling detail; compact landscape split; responsive navigation and safe areas. Preserve configurable identity, full-entry links, source-URL bookmarks and generated exports when resolving conflicts. | Audit fork-specific links, long content and many/uneven categories. Body text is 15px versus the planned 16px. Some controls have only 44px height, not 44 × 44 hit areas. |
| Swipes | One step on release; 36px minimum, 1.3:1 final-axis ratio, at most 1400ms; pointer capture, multi-pointer cancellation, client-coordinate detail drag. | These replace the draft 24px/1.5:1 proposal as imported behavior. No early axis lock or explicit resize/modal-open gesture cancellation. Verify browser edge gestures and hybrid input. |
| Boot, dialogs and settings | Reference boot stays 16:9 with a separate touch-size skip button; Visual Viewport sizes dialogs; focus explicitly follows tapped openers; fullscreen control is capability-gated. | Physical keyboard/browser-bar behavior, long identities, reading zoom and full accessibility review remain required. |
| Viewer | Explicit one-finger orbit and two-finger zoom/pan, compact controls, portrait canvas/framing, preserved requested pose on resize. | Visible directional pan and zoom alternatives are still missing. Real-device gesture/reset/close journeys remain required. |
| Rendering and resilience | Preserve quality defaults, stop updates while hidden, avoid rendering beneath the opaque 2D boot. | No measured physical Android/iPhone performance report for this fork, context-loss recovery, or complete failure/resume matrix. |
| Tests | Import viewport math/gesture checks and browser regression script; adapt the latter to variable category lengths and the available browser platform. | Upstream's reported iPhone trial and Windows WebKit screenshots are upstream evidence, not verification of this fork. Expanded CI, accessibility and physical-device checks remain open. |
| Offline PWA | Upstream additionally contains install/update UI, manifest, icons and a service-worker cache. Keep imported source/tooling as reference, but do not register the worker, advertise installation, or invoke its build step in this fork. | PWA stays outside this spec's scope. Its cache omits `reader.css`, `reader.js`, generated article/directory pages, KaTeX and content assets; enabling it needs a separate content-aware spec and validation. Upstream domain/deployment claims do not describe this fork. |

Integration work is limited to making the imported mobile implementation usable with this fork's content architecture and correcting integration regressions. The remaining plan gaps above are recorded for follow-up, not silently implemented as part of a rebase. The new fork verification report will record checks actually run.

### Acceptance criteria

- A touch-only user can enter or skip boot, navigate all categories and archives, read content, search, save/unsave, change settings, inspect/explode/reassemble/reset a model, and return to the same archive.
- At the specified viewport sizes, primary controls and text remain reachable with no unintended page-wide horizontal scrolling. Dialog close actions remain accessible with browser bars, notches and the software keyboard present.
- Rotation and viewport changes preserve selection, column memory, detail tab/scroll, active modal/search input, saved state, and viewer orientation/explode state. They do not reload the page or replay boot.
- Archive swipes and taps produce a single intended action; scrolling, multi-touch cancellation, dialog dismissal and viewer gestures cannot move the underlying archive.
- Desktop composition, baseline optics, looping/extraction rules, transition interruption, reduced motion, audio preferences, and generated reading URLs remain intact.
- Unsupported or failed interactive rendering leaves a working catalogue/article path. Explicit list mode and JavaScript-disabled generated reading remain usable.
- Required automated checks and representative real-device journeys pass, with performance measurements and any limitations recorded. Emulator-only testing is insufficient to claim the feature verified.

## Test plan

### Automated validation

- Run `npm run check` and `npm run build`; also build with `RHINELAB_CONTENT_DIR=examples/minimal` and, when available, the external content repository. Confirm generated output stays within UI `.generated/` and `dist/` and content remains unchanged.
- Add focused unit tests for viewport mode boundaries and gesture decisions: tap versus swipe, axis lock, threshold edges, cancellation, second pointers, and no duplicate dispatch. Exercise navigation with uneven categories, one archive, wraparound, and selection memory using the existing shared navigation logic.
- Add browser interaction coverage for mobile startup, explicit list mode, initialization failure, modal focus/isolation, touch actions, and resize preservation. Upstream adds `scripts/check-responsive.mjs`; adapt it for this fork's entry and variable content counts and provide reproducible invocation. Broader CI and device coverage remains pending. Test observable outcomes rather than checking for CSS strings.
- Check actual element bounds and document overflow at representative viewports, long/custom identity text, long titles, large content sets, and wide Markdown content. Confirm touch hit areas after all transforms.
- Capture desktop and compact archive, detail, dialogs and viewer screenshots with deterministic scene/reduced-motion settings where appropriate. Also exercise live transitions; frozen screenshots do not validate motion.

### Browser and device matrix

| Environment | Required checks |
| --- | --- |
| Phone emulation: 320 × 568, 390 × 844, 430 × 932 and their landscape sizes | Entry, layout, touch targets, scroll containment, all primary workflows. |
| Tablet emulation: 768 × 1024 and 1024 × 768 | Compact/desktop mode changes, touch controls on wide layouts, rotation and state continuity. |
| Desktop: 1920 × 1080, 1440 × 900, and boundary sizes around 1000 × 620 | Existing composition/input regression, no redirect on resize, no breakpoint flicker. |
| Physical iPhone / Safari | Full touch journey, safe areas, browser bars, software keyboard, browser zoom, audio unlock, viewer pinch/pan, background/resume. |
| Physical Android phone / Chrome, including a representative midrange device | Same journey, sustained navigation, rendering cost, context/resource behavior, asset failures. |
| Desktop Chromium, Firefox and WebKit where available | Shared state, fallback, keyboard/mouse input and browser integration. Emulated WebKit is not evidence of physical iOS GPU behavior. |

Record exact device, OS, browser version, viewport, pixel ratio, quality preset and commit for physical tests. Device availability is a validation dependency, not a reason to claim unrun checks passed.

### Manual regression journeys

1. Cold open, skip, replay, then navigate past both ends of uneven columns rapidly. Switch back to a column and verify its remembered selection. Repeat with touch buttons, swipes, mouse and keyboard.
2. Open detail during motion, return before entry completes, and reopen a returning card. Verify vertical extraction, clearance before rotation, turn-before-lower behavior, decryption continuity and desktop framing.
3. Read long content, change tabs, bookmark repeatedly, open/close the viewer, and rotate the device. Confirm the expected tab, text position and selected archive survive. Follow a generated article link and use browser Back.
4. Search with the software keyboard visible; scroll/filter/select results, close during entry, and reopen. Repeat saved/settings dialogs. Check keyboard focus, screen-reader names/order, 200% reading zoom, reduced motion, and background isolation throughout exit.
5. Orbit, pinch, pan, explode, toggle glass, reset and interrupt reset in the viewer. Close while loading or entering. Verify preserved detail state and no gesture leakage or stuck pointer capture.
6. Disable WebGL, fail a model request, simulate context loss, enter explicit list mode, and load generated reading with JavaScript disabled. Verify usable navigation and no redirect/retry loop. Test export and fullscreen capability feedback without requiring a desktop shortcut.
7. On each physical phone, record cold load and at least 60 seconds of repeated navigation, then repeat viewer open/close ten times. Record median and p95 frame times, actual buffer size and available resource observations. Initial performance target: median animation frame interval at most 33.3 ms and p95 at most 50 ms after load on the recorded representative devices, with no context loss or accumulating render loops. Misses require investigation and a documented resolution or scope revision before verification; these targets are not current measurements.

### Validation status

The upstream mobile foundation is integrated. Source comparison and fork validation are recorded in [MOBILE-SUPPORT.md](../verification/MOBILE-SUPPORT.md). Physical-device acceptance, screenshot comparisons and performance measurements remain incomplete; upstream reports are not verification of this fork.

For the initial documentation change, `npm run check`, `npm run build`, spec section/link checks, and `git diff --check` passed. The build reports its large JavaScript chunk warning. These checks validate the existing baseline and documentation, not mobile support.

## Change log

| Date | Change | Reason | Validation / outstanding work |
| --- | --- | --- | --- |
| 2026-09-09 | Created the first feature spec: responsive interactive archive, touch input, reading/dialog/viewer adaptation, fallback, staged landing and regression coverage. | The interactive UI is desktop-gated; mobile currently receives the reading catalogue. | Inspected entry, fixed-stage layout, pointer handling, viewer controls, render sizing, build/test scripts and repository constraints. Planning only; implementation and all mobile acceptance checks pending. |
| 2026-09-09 | Recorded planning-delivery validation and linked the spec-first rule, index and reusable template. | Establish a maintained specification before implementation. | Existing `npm run check` and `npm run build` passed; spec sections/links and whitespace checked. Build reports a large-chunk warning. Mobile acceptance remains untested. |
| 2026-09-09 | Rebase onto upstream `5abab02`; record mobile coverage, changed thresholds and remaining acceptance gaps. Plan entry/content/test compatibility fixes and defer PWA activation. | Reuse upstream mobile work while preserving the independent data architecture and original mobile scope. | Rebase conflicts resolved; initial checks found the settings test harness needs the new fullscreen/asset context. Browser and final checks pending. |
| 2026-09-09 | Reflow the fork-only READING INDEX link beneath compact system actions. | Browser bounds showed the extra link made navigation overlap the brand at 390 × 844; upstream has no such link. | Add overlap and reading-link bounds checks to the mobile regression; final rerun pending. |
| 2026-09-09 | Add portable browser-runner options, variable category-period checks, and state-based extraction waits; distinguish software/synthetic checks from native input validation. | Upstream tests assumed eight items, Windows Chrome, and fast rendering. | Core/quality checks and all three content builds passed; desktop workflow and catalogue fallback checks passed. Native mobile swipe and full-device acceptance remain unverified; see the fork report for final mobile results. |
| 2026-09-09 | Complete fork integration validation and retain In progress status for remaining acceptance work. | Import the usable upstream foundation without claiming the full original plan is complete. | Core and quality checks, default/minimal/external builds, four reading fallback cases, desktop workflow and three mobile viewport workflows passed. Mobile archive swipes were synthetic under test-only software rendering/reduced motion; native timing, screenshots and real-device performance remain unverified. Evidence: [fork report](../verification/MOBILE-SUPPORT.md). |
