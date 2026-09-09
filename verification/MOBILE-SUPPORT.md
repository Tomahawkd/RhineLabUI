# Mobile integration in this fork

Date: 2026-09-09. Upstream base: `5abab02`. Pre-rebase fork: `42572a4`.

This report covers the integration of upstream mobile support with the Markdown-driven personal-site fork. The original requirements, comparison and remaining work are maintained in [spec 0001](../specs/0001-mobile-support.md#upstream-integration-comparison--2026-09-09). [RESPONSIVE.md](RESPONSIVE.md) is upstream evidence and must not be treated as a test run of this fork.

## Integration

- Rebased the fork's content architecture and spec-first workflow onto the upstream mobile commit. Recovery branch: `backup/main-before-mobile-rebase-20260909`.
- Resolved conflicts in `DESIGN.md`, `index.html`, `package.json`, and `src/main.ts`, retaining configurable identity, content generation, source-URL bookmarks, full-entry links and generated TXT exports.
- Enabled responsive startup by removing the old small-screen gate and resize redirect. Explicit `/?view=list`, missing-WebGL and initialization fallback remain available.
- Kept the upstream responsive layout, camera-only reframing, swipe recognizer, viewer gestures, safe areas, modal sizing, and quality defaults. The fork-only READING INDEX link now sits below compact system actions: measured original bounds at 390 × 844 overlapped the brand (brand x=20–133.4; navigation x=59.2–370).
- Imported upstream viewport checks into `npm run check`. Added `npm run check:responsive` and adapted its browser launch for Linux/optional channels, category-period checks for variable data, and system-nav overlap checks.
- PWA source and tools remain in history and the tree, but production does not import the PWA runtime, advertise the manifest/install UI, or generate/register a service worker. Upstream cache assumptions do not cover generated reading content. No data-site deployment pin is changed by this rebase.

## Validation

- `npm run check`: passed, including content, ordering, configurable identity, motion, appearance, assembly and upstream viewport checks.
- `node scripts/check-quality.mjs`: passed.
- Default `npm run build`: passed (40 records / 5 categories / 45 pages).
- `RHINELAB_CONTENT_DIR=examples/minimal npm run build`: passed (3 records / 2 categories / 5 pages).
- `RHINELAB_CONTENT_DIR=../tomahawkd.github.io npm run build`: passed (32 records / 5 categories / 21 pages); external repository status remained clean. Default output was rebuilt afterward.
- Builds retain the existing large-JavaScript-chunk warning.
- Mobile entry/fallback probe: passed at 390 × 844 for explicit list mode (including an article response), missing WebGL, disabled JavaScript, and an aborted interactive import. Local probe: `.tools/check-mobile-entry.mjs`.
- Browser workflows passed: desktop 1920 × 1080; landscape 844 × 390; portrait 390 × 844; small portrait 320 × 568. Covered bounds/navigation-brand separation, full category-period looping, extraction 4.05 and decryption, bookmark scroll continuity, viewer explode/assemble, pinch/pan, orientation/selection preservation, dialogs/search and focus return. Completed cases reported no page errors.
- Desktop evidence is retained in `.tools/responsive/desktop-pass-native-swipe-failure.json` (that run subsequently failed its native landscape swipe). The successful mobile rerun is `.tools/responsive/regression-chromium.json`, with `passed: true` for all three cases. Mobile archive swipes use synthetic pointers; viewer multi-touch uses Chromium injection. All runs use software rendering with lower test settings and reduced motion. Screenshots and physical-device/native archive swipe timing remain unverified.

## Reproducing browser checks

Run `npm run dev -- --port 5187` in one terminal, then `npm run check:responsive` in another with a locally installed Playwright and Chromium browser. The script accepts `PLAYWRIGHT_MODULE` (absolute module path), `REVIEW_URL`, `REVIEW_CHANNEL`, `REVIEW_ENGINE`, and a comma-separated `REVIEW_CASES`. The cases include desktop, laptop, wide, ultrawide, tablet, landscape, portrait, small and short-landscape.

This session uses the available Playwright module in `/tmp/logger-browser/node_modules/playwright/index.mjs`, Chromium binaries under `/tmp/logger-browser/browsers`, and Linux libraries under `/tmp/logger-browser/libs/usr/lib/x86_64-linux-gnu`. These are environment-specific paths, not dependencies added to the application.

For software-only environments, `REVIEW_SOFTWARE=1` selects SwiftShader, test-only reduced motion, 0.5 device pixel ratio and lower render settings; `REVIEW_DPR` can override the test pixel ratio (the final run uses 0.25). `REVIEW_SCREENSHOTS=0` skips screenshot capture. Software runs use synchronous synthetic archive pointer events, as the 1.4-second production swipe deadline cannot be reliably exercised with this slow renderer. Viewer gestures still use Chromium touch injection. Native archive timing/hit testing is not covered by the synthetic path. Neither option changes production defaults. The final mobile invocation selected `REVIEW_CASES=landscape,portrait,small`, with `REVIEW_SOFTWARE=1 REVIEW_DPR=0.25 REVIEW_SCREENSHOTS=0`. Reports are written to `.tools/responsive/regression-chromium.json`; screenshots, when enabled, go to the same directory. Passing with these options is interaction evidence only, not baseline visual fidelity, full-motion validation or physical-device performance evidence.

The first desktop run timed out waiting for preview extraction under software rendering. A portrait retry reached the archive with no page errors, but screenshot capture timed out. These attempts do not count as full passes. A subsequent fixed-delay assertion observed extraction at 3.93 before it reached 4.05; the runner now waits for actual extraction and inspection readiness. The reduced-motion interaction run and separate fallback probe are reported separately.

## Remaining acceptance work

The upstream base provides much of the planned mobile functionality, but spec 0001 is still **In progress**. In particular:

- Body text is 15px rather than the planned 16px, and not all controls meet a 44 × 44 target.
- Viewer pan/zoom still relies on gestures or keyboard; visible alternatives remain to be added.
- Gestures classify the axis on release; explicit orientation/modal-open cancellation and early axis lock remain open.
- Physical iPhone/Safari and Android checks, software keyboard/browser bars, screen-reader/zoom review, long/custom content, and measured performance remain unverified for this fork.
- Runtime context-loss recovery and the full asset-failure/background-resume matrix remain open.
- No new cross-device pixel comparison or physical-device frame-time report has been produced here.
