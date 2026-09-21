# dsh-paste-input

[简体中文](./README.md) | **English**

A file-input enhancement plugin for the DSH WebUI: **Ctrl+V paste** + **whole-page drag & drop** + **select files/folders**. On send, files are copied into the session workspace's temporary attachments directory, and the attachment text blocks in conversation bubbles are **collapsed into file chips**.

Derived from [dsh-external/dsh-multimedia-webui-input](https://github.com/dsh-external/dsh-multimedia-webui-input) (MIT), adding clipboard paste input, a first-use notice dialog, and bubble attachment collapsing on top of it.

> **Pick the plugin version that matches your DSH** (a mismatch crashes: common symptom `useConversation is not a function`)
> - DSH **0.1.6-alpha.2** (npm latest): install the **new** version (the default command below)
> - DSH **0.1.5-x / 0.1.2-alpha.x / 0.1.2-rc.1**: install the **old** version `'@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.27'`
> - DSH **0.1.1-rc.2** (old npm): install the **oldest** version `'@dsh-external/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.5'`

## Installation (profile mode)

```sh
# Option 1: pinned-tag git dependency (public mirror, recommended; github:lhh010/dsh-paste-input also works)
dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.28'

# Option 2: local link
# dsh plugin --profile web add link:/path/to/dsh-paste-input
```

Append to `~/.dsh/profiles/web/cordis.patch.yml` (hot-reloaded, no restart needed):

```yaml
- insert:
    - id: dsh-paste-input
      name: '@dsh-community/dsh-paste-input'
```

> **Install tips**: pnpm 11 may block node-pty build scripts on first install — run `pnpm approve-builds --all` under `~/.dsh/profiles/web` and re-run the install; then **hard-refresh the browser** (Ctrl/Cmd+Shift+R).

### 2026-09-20 · v0.1.28 — Declare dsh.bundle layer + real-version update check

- **New `cordis.patch.yml`**: the plugin ships its own bundle patch file, declared via `package.json → dsh.bundle.patch`; the host applies it automatically at the bundle layer, so users no longer need to manually maintain `~/.dsh/profiles/web/cordis.patch.yml`
- **Version check upgrade**: no longer relies solely on a hardcoded `PLUGIN_VERSION` constant; on startup the check reads the host-loaded plugin version from the `__DSH_BOOT__` manifest (with a dev-mode `rev` fallback), comparing against the remote latest tag using the real running version — avoids false-positive update prompts during local development or un-published builds
- **Sync**: install commands now point to `#v0.1.28`

### 2026-09-17 · v0.1.27 — Adapt to dsh-v0.1.6-alpha.2

Adapted to dsh 0.1.6-alpha.2 multi-instance refactor: the current session is now resolved via `uiSession.current` (`{ key, ctx }`) (the `sessions.list.current` field from alpha.1 was removed, previously causing paste/drop/attach-button to all show "please open a session first"); paste, drag, and attach-button paths all switched, with alpha.1 fallback retained. lib node --check all green, alpha.2 real-host verified: chips restored.

### 2026-09-15 · v0.1.26 — Declare support for dsh-v0.1.6-alpha.1

Declared support for dsh-v0.1.6-alpha.1 (published on npm, pinned-version real-host verified; zero code delta on this plugin's client surface, lib node --check all green, loads normally on real host).

### 2026-09-10 · v0.1.25 — Declare dsh-v0.1.5-rc.2 compatibility

- **Verification**: rc.2 has no client-plugin-facing changes and needs no code change; loaded and confirmed on a real rc.2 host (tag fb2c4b9e); paste-to-input box, hover preview, and viewer all work

### 2026-09-10 · v0.1.25 — Declare support for dsh-v0.1.5-rc.1

- **Verification**: rc.1 is the first 0.1.5 release candidate, zero code delta on this plugin's client surface; published on npm, pinned-version real-host verified; baked PLUGIN_VERSION constant synced

### 2026-09-09 · v0.1.24 — Fix folding of legacy end-marker messages

- **Fix**: session history contains two end-marker spellings (current `==== END DSH_PASTE_INPUT ====` vs the V1-suffixed variant written by older cached bundles); the parser only accepted the current form, so legacy messages never folded. Both spellings are now accepted. Note: the V1 spelling is legacy-only (written by very early bundles only) and **may be dropped in a future release**

### 2026-09-09 · v0.1.23 — Declare support for dsh-v0.1.5-alpha.2

- **Verification**: alpha.2 changes are sidebar document preview, model file delivery, minimal default tool adjustments, and `fs-ext` install fix — zero code delta on this plugin's client surface; published on npm, pinned-version real-host verified; baked PLUGIN_VERSION constant synced

### 2026-09-08 · v0.1.22 — Declare support for dsh-v0.1.5-alpha.1

- **Verification**: 0.1.5 changes are session format V3 / `ctx.agent` removal / host client-bundle service route change to `/plugins/??` composite routing — zero code delta on this plugin's client surface; published on npm, pinned-version real-host verified, no code changes needed

### 2026-09-05 · v0.1.21 — Declare support for dsh-v0.1.3-alpha.2

- **Verification**: alpha.2 changes are all in pi-ai / Web top bar / subagent messages / host — zero code delta on this plugin's client surface; published on npm, pinned-version real-host verified, no code changes needed

### 2026-09-05 · v0.1.20 — Paste-record persistence toggle (sessionStorage, default off)

- **New**: a clipboard-record persistence toggle now sits next to the "Profiles" button in the title bar (default **off**). When on, paste records mirror to `sessionStorage` — after a page reload, leftover paste-reference chips in the composer can still be sent (previously: "Attachment selection is no longer available in this browser tab").
- **Limits**: files over 1 MiB are not persisted; the snapshot caps at roughly 3 MiB total (oversized records are skipped).
- Turning the toggle off clears the persisted records.
- **Granularity**: persistence is decided per paste record at creation time (toggle on + file within limits). Switching off clears already-persisted records; re-enabling does not backfill old ones; sent messages are unaffected (files already live host-side).

### 2026-09-04 · v0.1.19 — Declare support for dsh-v0.1.3-alpha.1

- **Verification**: 0.1.3 breaking changes are on host/session side (SessionHandle / session format v2), composer/input surface unaffected in practice; npm not published, source real-host verified (paste-to-input, hover preview, viewer all work), no code changes needed

### 2026-09-03 · v0.1.18 — Declare support for dsh-v0.1.2-rc.1

- **Verification**: alpha.5 → rc.1 is version-bump-only upstream (252 files, zero code diff); verified live on rc.1 (hover preview / viewer working), no code changes needed

### 2026-09-03 · v0.1.17 — Image/GIF hover preview + click viewer (zoom & pan)

- **New (hover thumbnail)**: image attachments (png/jpg/jpeg/gif/webp/bmp/avif/ico) pop a small preview card on chip hover — animated GIFs play as-is. Works on both the composer's pending chips (local bytes via blob URL) and the bubble's sent chips (host reads the file back after ownership-marker validation)
- **New (click viewer)**: clicking an image chip opens a fullscreen viewer — cursor-centered wheel zoom (20%–800%), left-drag panning, double-click toggles 1×/2×, `+`/`-`/`0`/`Esc` shortcuts, and a toolbar with the zoom percentage, reset, copy-full-path, and close; GIFs keep playing in the viewer
- **New host route (read-only)** `GET /dsh-paste-input/v1/file?root=<send dir>&path=<relative path>`: serves only image files **declared in the send's ownership marker** (`.dsh-paste-input.json`; SVG excluded to avoid same-origin script execution), with path resolution confined to the send directory and a 64 MiB per-file cap
- Non-image chips keep their behavior (hover shows the raw attachment block, click copies the path); the image chip's copy-path action moves into the viewer toolbar
- **Fix (dock chip crash, latent since v0.1.16)**: the composer dock chip's remove button referenced `busy`, a variable from other components' scope — the moment a chip rendered it threw a ReferenceError and the error boundary swallowed the whole dock slot (symptom: the dock's attachment chips vanish); the dangling reference is removed

### 2026-09-02 · v0.1.16 — Fix AttachButton crash + version check moves to jsdelivr

- **Fix (AttachButton crash)**: the `conversation.input.left` slot provides no owner props (no `input`), so `props.input.phase` read undefined and crashed the slot entry. Now uses optional chaining with a `'plain'` default (`add()` has its own phase guard). The dock slot's `occurrences` gets the same defensive default
- **Fix (403 spam)**: version-check tag source moved from `api.github.com` (rate-limited, 403s unauthenticated) to `data.jsdelivr.com/v1/packages/gh/` (CDN, no rate limit, CORS-friendly)

### 2026-09-02 · v0.1.15 — Declare support for dsh-v0.1.2-alpha.5

- **Verification**: alpha.5 is a pure bug fix (upgrade path issue), no change to client runtime API; lib output verified

### 2026-09-02 · v0.1.14 — Declare support for dsh-v0.1.2-alpha.4

- **Verification**: alpha.4 has no destructive changes to the client runtime API (changelog is host-side Session events refactor only); lib output verified

### 2026-09-02 · v0.1.13 — Update prompt gains version routing + troubleshooting

- **Fix (update prompt)**: step 0 added (run `dsh --version` first and pick the tag per the README compatibility table) and step 3 (on install failures / version mismatch / startup errors, consult the README compatibility and known-limitations sections first); the original install steps are unchanged

### 2026-09-01 · v0.1.12 — Version check caching / 403 fallback

- **Fix (403 spam)**: when the GitHub tags API returns 403 (rate-limited / unauthenticated), the old code re-fetched on every page load and retry, flooding the console. Verdicts are now cached in localStorage per status: OK for 10 minutes, transient network failure 60s, hard 403 for 5 minutes — inside the window the check returns the cached verdict without touching the network; a manual "retry" still forces one fetch
- **Fallback copy**: distinguishes "network unreachable" from "GitHub refused (rate-limit / 403)" — the latter shows "version check temporarily unavailable (GitHub refused), cached" instead of misattributing it to the network

### 2026-09-01 · v0.1.11 — Version chip no longer fooled by GitHub CDN lag

- **Fix**: in the minutes right after pushing a new tag, the GitHub tags API / raw CDN still serve the old tag, and the "already latest" chip displayed that stale REMOTE tag as the latest (e.g. running 0.1.10 while showing "latest 0.1.9"). It now shows whichever of the fetched tag and the running version is newer; the offline chip's retry path applies the same rule

### 2026-09-01 · v0.1.10 — Fix broken dock removal + same-name auto-numbering

- **Fix (removal → unavailable)**: on DSH 0.1.2-alpha the input machine's occurrence offset/length are **clipboard-projection coordinates** (a chip spans its full `[attachment: …]` text) while `consumeToken`'s span guard works in **detect-projection coordinates** (a chip is exactly one U+FFFC character) — the old code passed clipboard offsets straight through, the replace always failed, and the already-deleted record left the chip showing "unavailable" in the dock and resident in the composer. The span is now converted to detect coordinates (each earlier chip contributes length−1 fewer characters), with a setDraft whole-range slice fallback
- **Fix (second paste errored / displaced the first)**: `insertReference`'s insertion point used `snapshot.draft.length` (clipboard-projection length); once a first attachment existed the point fell past the detect text and `The DSH composer changed before the attachment could be inserted` fired. The point is now folded into detect coordinates the same way, so consecutive pastes coexist
- **Added (unified paste renaming)**: pasted files get a unified base name — images `paste_image.<ext>`, other files `paste_file.<ext>` (extension from the original name, falling back to the MIME map); collisions append `(2)`, `(3)`… (conflict set = live composer chips + records), and the rename carries into the upload path. **Only the paste path renames**; drops and the file/folder picker keep their real names
- **Verification**: verified live — two consecutive screenshot pastes yield `paste_image.png` and `paste_image(2).png` side by side; dock × removal clears both views; node --check clean

### 2026-08-20 · v0.1.5 — Declare DSH 0.1.1-rc.1 compatibility (real boot verification)

- **Verification**: real boot verification passed on DSH npm `0.1.1-rc.1` — the boot manifest includes this plugin and client.js returns 200; the v0.1.4 rc.8 adaptation (`consumeToken` whole-range removal, official `appearance: 'file'`, whole-chip edit protection) shows no regression on 0.1.1-rc.1 (the `inputTriggers.registerSource`, `conversation.input.for` facade, and the `conversation.input.left/dock` and `settings.section` slots are unchanged)

### 2026-08-20 · v0.1.4 — DSH 0.1.0-rc.8 adaptation (broken removal + inline chip appearance)

- **Fix (broken removal)**: rc.8's input machine gives a reference occurrence the full inline range of `@` + label (no longer 1 placeholder character); v0.1.3's dock × click deleted only the `@` character, leaving attachment text behind in the composer. Now uses rc.8's official removal verb `input.consumeToken` (span CAS whole-range removal), falling back to `setDraft` slicing by `occurrence.length` on older hosts
- **Fix (inline chip appearance)**: removed the hand-rolled `📎 ` emoji prefix and 8-character truncation (rendered on rc.8 as "blue @ + paperclip + blue filename"); the label is now the bare file name with the official `appearance: 'file'`, matching the official `@file` reference chip; the full path and size remain in the dock chip above the composer
- **Added (whole-chip edit protection)**: the inline chip's file name can no longer be edited in place (partial edits are blocked and the whole chip is auto-selected; the next keypress deletes or replaces it entirely) — see "Whole-chip edit protection" above
- **Verification**: verified in practice on DSH npm `0.1.0-rc.8` — paste/drop → dock chip → × removal clears the composer in sync; send → copy into the attachments directory → bubble chip collapsing all work

### 2026-08-13 · v0.1.3 — Final snapshot service rename migration (snapshot0812 + npm rc.5)

- **Migration (client)**: `slash` → `inputTriggers` (4 places in lib/client.js: inject arrays ×2 + the `ctx.get` + the `registerSource` call); the `dsh.client` metadata inject migrated from `@deepseek-ai/dsh-client-ui-slash` to `@deepseek-ai/dsh-client-ui-input-trigger` — the final snapshot renamed the input-trigger service together with the official package; the service and the `registerSource` API are unchanged
- **Migration (host)**: `httpServer` → `webServer` (2 places in lib/index.js: the inject array + the `ctx.webServer.register` call) — the final snapshot renamed the host-side HTTP route service; the `register({ kind: 'prefix', path, handler })` API is unchanged
- **Verification**: real boot verification passed on the DSH final snapshot (`snapshots/20260812T172954Z-final`) and the npm rc.5 (`@deepseek-ai/dsh@0.0.1-rc.5`) consumer (boot manifest includes this plugin, client.js returns 200, webServer upload route loads successfully)

### 2026-08-11 · v0.1.2 — Client plugin metadata migration (snapshot0810)

- **Migration**: package.json migrated from the top-level `dshClient` declaration to the nested `dsh.client` (inject preserved as-is) — 0810's ClientModuleHostService only reads `pkg.dsh.client`; the old field is silently ignored and the plugin does not enter the boot graph
- **Verification**: real verification passed on DSH snapshot0810 (full chain: paste → copy into the attachments directory → bubble chip collapsing)

### 2026-08-10 · v0.1.1 — Fix misplacement of bubble-collapsed chips

- **Fix**: when text is typed both before and after chips on send (especially multi-file sends), the collapsed file chips were mispositioned — previously all user text was merged into a single text block piled at the top, the first chip floated to the right of the first line of text due to the flex layout, and the remaining chips scattered below the text block; now they render interleaved in source order (text → chip → text → chip…), each text segment occupies its own line, and chips of adjacent attachment blocks line up side by side automatically
- **Fix**: the collapsed-area text and chips now align with the bubble's internal 16px text indent (removing the previous extra horizontal inset and bottom gap)
- **Verification**: real verification passed on DSH snapshot0809

## Features

- **Ctrl+V paste**: paste a screenshot / copied image / file → added to the input box as an attachment (a notice dialog pops up on first paste; you can check "Don't show again", and the choice persists in browser localStorage)
- **Whole-page drag & drop**: drag files/folders to anywhere on the page (chat area, blank space, input box) to add them as attachments; dragging text/links keeps the browser's default behavior
- **Select**: the paperclip button on the left of the input box → select files / select folders
- **Bubble collapsing**: after sending, the verbose attachment-path text block in the message bubble (carrying the `==== DSH_PASTE_INPUT_V1 ====` marker protocol) is automatically collapsed into a 📎 file chip; text you typed before and after the chip is preserved interleaved in original order (on multi-file sends, text and each file's chip alternate segment by segment, with each chip on its own line); hovering the chip shows the complete original attachment block (paths/manifest/file list), and clicking the chip copies the full path
- **Image preview**: image/animated-GIF attachments pop a small thumbnail on chip hover (GIFs play as-is), and clicking the chip opens a fullscreen viewer — cursor-centered wheel zoom (20%–800%), drag panning, double-click 1×/2×, `+`/`-`/`0`/`Esc` shortcuts, copy-full-path in the toolbar. Both composer chips and sent-bubble chips are supported (the host reads sent files back after ownership-marker validation)
- On send, files are copied to `<session workspace>/.dsh/tmp/attachments/<session>/<send>/`, and the absolute paths are prefixed to the message for the model — no permission issues
- Settings panel: attachment usage statistics and cleanup per session/workspace (protected by ownership markers, with double confirmation)

## Working with dsh-vision: screenshot recognition

Together with the [dsh-external/dsh-vision](https://github.com/dsh-external/dsh-vision) plugin (which registers the `view_image` tool and bridges any OpenAI-compatible VLM, defaulting to Zhipu's free `glm-4.6v-flash`), screenshots pasted or dragged into this plugin can be **recognized directly**:

1. Take a screenshot (Win+Shift+S) → paste it or drag it into DSH
2. After sending, the screenshot is copied into the workspace attachments directory
3. The model sees the attachment path → calls `view_image` → the VLM returns the image content (OCR text extraction, reading charts, recognizing UI layouts, etc.)

The two plugins are zero-coupled: this plugin handles "getting files into the conversation", dsh-vision handles "looking at images", and they connect through the workspace attachment paths.

## Attachment message protocol

Attachment blocks are delimited by an explicit marker (model-visible text, used by bubble collapsing for recognition):

```
==== DSH_PASTE_INPUT_V1 ====
<absolute path of the attachments root directory>

Files: N
Manifest: .dsh-paste-input.json
Attached files (paths are relative to the root above):
- "file.txt" (2.0 KiB)
==== END DSH_PASTE_INPUT ====
```

Only the marked format is supported (historical unmarked messages are not collapsed). A blank line is placed before and after the marker so that user-typed text and the marker never share a line.

## Limitations

- Support for pasted files varies by browser: **Chrome/Edge** only expose media such as images (screenshots, copied images) and text/HTML in the paste event — pasting after "copying files" from the file manager produces no file entries; **Firefox** supports pasting files, but likewise **does not provide absolute paths**. Browsers never expose local file paths to web pages for security reasons, so pasted content is always stored with the `filename` as a relative path — for scenarios that need the original path, use **drag & drop** or the **select files/folders** button
- Single file ≤ 1 GiB, per batch ≤ 2 GiB, ≤ 10000 files, ≤ 64 levels

## Prompt install (let DSH install it)

Paste this prompt into any DSH session and the agent installs it for you:

> Install the dsh-paste-input plugin (DSH file-input enhancement plugin (paste/drag files)):
> 1. Run `dsh plugin --profile web add '@dsh-community/dsh-paste-input@github:lhh010/dsh-paste-input#v0.1.28'` (the first run may fail because pnpm 11 blocks node-pty build scripts)
> 2. Under `~/.dsh/profiles/web`, run `pnpm approve-builds --all` (approve the build scripts)
> 3. Re-run the install command from step 1
> 4. Remind me to hard-refresh the browser (Ctrl/Cmd+Shift+R)
> On errors, first check the FAQ/known limitations in the README at <https://github.com/lhh010/dsh-paste-input>.

## License

MIT (includes the derivation notice for dsh-multimedia-webui-input)
