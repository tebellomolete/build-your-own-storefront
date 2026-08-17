# Dev Coffee — Build Your Own Storefront

Shopify Theme Development Module · Day 1 · Store Setup & Development Environment

**Change notes.**

1. **Theme migration.** Originally attempted with Shopify Dawn on a store named `my-store-yofcpcr4`. Per instructor guidance, migrated to Horizon.
2. **Store rebuild.** The original store carried leftover schema data from prior themes that produced persistent upload errors on Horizon 4.1.3. Fixed by provisioning a clean store (`dev-coffee-46ztlwtt.myshopify.com`) with no theme history — CLI now uploads cleanly.
3. **Brand direction.** The niche stayed as specialty coffee, but the brand was retooled from a generalist audience (home baristas) toward developers — the training programme is a developer programme, and the audience shift makes the store a stronger portfolio piece by the end of the module.

## Store Brief

### The niche: specialty coffee, developer-oriented

`dev.coffee()` — trading as **Dev Coffee** — is an independent specialty coffee storefront selling roasted whole beans, pre-ground coffee, and pour-over equipment to software developers. The catalogue is deliberately variant-heavy: every bean product exposes roast level, grind (Whole Bean / Drip / Espresso / French Press / AeroPress), bag size (250g / 500g / 1kg), and purchase type (one-time vs. subscription, at monthly or quarterly cadence). Origin, altitude, processing method, and tasting notes are pinned as structured attributes rather than free-text fields. Equipment carries its own variant dimensions.

This complexity is deliberate. Later this week the module introduces advanced filtering and dynamic metafields, and a niche whose products only had one or two variants wouldn't stretch either feature. Coffee gives filtering something real to filter _by_ — roast, origin, brew method, subscription cadence, price band — and gives metafields somewhere real to live: tasting notes, altitude, process, brew ratios, and dev-themed use-case tags (`debugging`, `long-meetings`, `pair-programming`) don't fit into Shopify's default product fields, so they're the exact use case Metaobjects and Metafields were designed for.

The choice is explicitly not the "outdoor & heritage lifestyle goods" niche used in the instructor's Northfield Supply Co. demo.

### Target audience

Dev Coffee is aimed at software developers who take coffee seriously enough to buy specialty beans but don't have time to shop them the way a full-time home barista would. They read Hacker News in the morning, work through a subscription model naturally because they already pay for six SaaS tools, and they respond to product copy that treats them as insiders — dev-themed product names (`Debug & Brew`, `null pointer`, `Runtime Error`, `Async / Await`) signal that the brand is written _by_ someone who understands the audience, not _at_ them.

## Page Scope

Three custom pages will be built in addition to the standard homepage, product, collection, cart, checkout, and policy templates. Each is backed by a Shopify **Metaobject** — a custom content type with defined fields — so the content is structured, reusable, and editable through the admin without touching Liquid.

### 1. Brew Log

A brewing tutorials hub. Each entry is a full recipe for a specific method — pour-over, AeroPress, French press, moka pot, espresso, cold brew — with steps written in a deliberately code-flavoured voice ("init the kettle", "configure the grind", "deploy hot water").

**Metaobject: `brew_recipe`** — fields: `method_name` (single-line), `difficulty` (single-line: beginner / intermediate / advanced), `total_time_minutes` (integer), `coffee_water_ratio` (single-line, e.g. "1:16"), `ideal_grind` (single-line), `equipment` (list of product references), `steps` (multi-line rich text), `hero_image` (file reference).

### 2. The Beans

Profile pages for each single-origin bean and each blend — where it's grown, altitude, processing method, tasting notes, and which brew method it plays best with. Doubles as SEO landing pages for origin-specific search queries.

**Metaobject: `bean_profile`** — fields: `origin_country` (single-line), `region` (single-line), `altitude_range` (single-line, e.g. "1,600–2,100m"), `process` (single-line: washed / natural / honey), `roast_level` (single-line), `flavour_notes` (multi-line text), `ideal_brew_methods` (list, e.g. pour-over / espresso), `hero_image` (file reference), `associated_products` (list of product references).

### 3. FAQ

A structured FAQ where each question-and-answer is its own metaobject entry rather than being trapped inside a single hardcoded Liquid template. Categorised so the page can filter by topic (Subscriptions, Shipping, Brewing, Equipment).

**Metaobject: `faq_entry`** — fields: `question` (single-line), `answer` (multi-line rich text), `category` (single-line, enum-style: subscriptions / shipping / brewing / equipment / general), `sort_order` (integer), `is_featured` (true/false).

## Dev Environment

- **Development store:** `dev-coffee-46ztlwtt.myshopify.com` (display name "Dev Coffee")
- **Store currency:** ZAR (South African Rand)
- **Shopify CLI version:** 4.6.1 (verified with `shopify version`)
- **Theme:** Horizon 4.1.3, installed via the Shopify Theme Library and pulled locally with `shopify theme pull`. A direct GitHub clone of Horizon's `main` branch was tried first but produced schema-validation errors on upload — the theme-store install is the validated stable version.
- **Local preview:** `http://127.0.0.1:9292`
- **GitHub repository:** `github.com/tebellomolete/build-your-own-storefront`. Horizon files committed to `main`.
- **Catalogue:** the default Shopify placeholder products were deleted and replaced with a six-product dev-themed coffee catalogue: `Debug & Brew`, `null pointer`, `Runtime Error`, `Async / Await` (beans, with Grind and — on the flagships — Size variants), plus `The Compiler` (pour-over dripper) and `git push` (paper filters). All six products are grouped under the collection `./roast`.

**Hot reloading verified.** With `shopify theme dev` running, a temporary `<h1>TEST</h1>` was added to `layout/theme.liquid` immediately below the `<body>` tag. The browser at `http://127.0.0.1:9292` reloaded automatically and displayed the tag without a manual refresh. The tag was then removed, and the browser reloaded again to confirm the change had propagated back out.

## Stretch A — GitHub sync (Online Store admin)

Connected `github.com/tebellomolete/build-your-own-storefront` (branch `main`) to the Dev Coffee store's theme library via Online Store → Themes → Add theme → Connect from GitHub.

**Local CLI edit vs. Theme Editor edit when GitHub sync is active.** A change made through `shopify theme dev` (or a commit pushed from local) flows _outward_: the developer edits Liquid or JSON in their IDE, commits, pushes to `main`, and Shopify's sync pulls the change into the connected theme in the library. Git history is authoritative — every change is a commit with an author, message, and diff. A change made through the Shopify Theme Editor (drag a section, tweak a colour, edit copy in the customizer) flows _inward_: the merchant edits in the browser, Shopify auto-commits the resulting `settings_data.json` (and sometimes template JSON) mutation directly to the connected branch on GitHub, authored as the Shopify GitHub bot. This means the branch has two writers, and any developer pulling `main` locally needs to expect merchant commits interleaved with their own. The practical rule: developers work in `sections/`, `blocks/`, `snippets/`, `layout/`, `assets/`, and `config/settings_schema.json`; merchants effectively "own" `config/settings_data.json` and any JSON template edits done through the customizer. Conflicts arise when both sides touch the same file, which is why keeping developer work in `.liquid` code and merchant work in the customizer is a discipline worth enforcing even though nothing in Shopify technically prevents crossover.

## Stretch B — VS Code Liquid extension + format on save

Installed the official Shopify Liquid extension (`Shopify.theme-check-vscode`) and configured `.vscode/settings.json` to auto-format `.liquid` files on save:

json
{
"[liquid]": {
"editor.defaultFormatter": "Shopify.theme-check-vscode",
"editor.formatOnSave": true
}
}

The `[liquid]` selector scopes the settings to Liquid files only, so JavaScript and JSON files continue to use their own default formatters. `editor.defaultFormatter` names the extension that should handle the formatting, and `editor.formatOnSave` triggers it on every `Cmd+S`.

---

# Assignment 1.2 — Liquid Fundamentals

Shopify Theme Development Module · Day 2 · Editing Sections, Snippets & Filters

Horizon 4.1.3 is blocks-first. There is no `sections/main-product.liquid` monolith to edit — the product page is `sections/product-information.liquid`, which renders individual blocks (`blocks/price.liquid`, `blocks/product-title.liquid`, `blocks/product-description.liquid`, `blocks/buy-buttons.liquid`, and so on). The collection page uses `sections/main-collection.liquid` and renders each product through `blocks/_product-card.liquid`, which delegates to `snippets/product-card.liquid`. Because Horizon's shipped blocks upgrade on theme updates, this assignment's edits go into **new** blocks (`blocks/coffee-details.liquid`, `blocks/coffee-card-unit-price.liquid`) and a **new** snippet (`snippets/coffee-unit-price.liquid`) rather than forking Horizon's stock files.

## Filter Log

Five distinct filters, each authored in files added this assignment. Every filter's effect is described in terms of Dev Coffee's real data (ZAR pricing, gram weights, dev-themed product copy) — not the generic Shopify filter behaviour.

| Filter       | Target file                         | What it changes on the page                                                                                                                                                                                        |
| ------------ | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `money`      | `blocks/coffee-details.liquid`      | Formats the savings amount on the product-page badge — e.g. `compare_at_price − price = 4000` cents renders as `R40.00` next to "On sale" instead of the raw integer.                                              |
| `image_url`  | `blocks/coffee-details.liquid`      | Requests a 400-px-wide render of `product.featured_image` for the badge-panel thumbnail on the product page, so the CDN serves a small file instead of the full-resolution roast shot.                             |
| `truncate`   | `blocks/coffee-details.liquid`      | Caps the hardcoded brew-notes placeholder (Stretch B) at 90 characters on the product page, so the tasting-note line never wraps past the badge panel.                                                             |
| `strip_html` | `blocks/coffee-details.liquid`      | Removes the wrapping `<p>` tags from the brew-notes placeholder before `truncate` runs, so the character budget isn't wasted on markup and no raw tags leak into the rendered output.                              |
| `divided_by` | `snippets/coffee-unit-price.liquid` | Divides `variant.price × 100.0` by `variant.weight` (grams) to produce a per-100g price for each coffee bag — makes 250g vs 500g vs 1kg bags directly comparable on both the product page and the collection card. |

Distribution: `money`, `image_url`, `truncate`, and `strip_html` are exercised on the product page via `blocks/coffee-details.liquid`. `divided_by` is exercised on the collection card via `blocks/coffee-card-unit-price.liquid`, which renders `snippets/coffee-unit-price.liquid`. The same snippet is also rendered from `blocks/coffee-details.liquid`, so `money` and `divided_by` both run on the product page too — the total of five distinct filter _names_ is what the assignment counts.

## Conditional Logic

**Object property.** `closest.product.selected_or_first_available_variant.compare_at_price` compared to `.price`. This is the same field Shopify's admin exposes as "Compare-at price" — a variant-level number in cents, non-blank only when the merchant has set a strike-through price.

**File.** `blocks/coffee-details.liquid`, wrapping the "badge" paragraph.

**Branches.**

- **True** (`compare_at_price != blank` and `compare_at_price > price`): renders a red "On sale — you save `{{ savings | money }}`" badge, where `savings = compare_at | minus: price`.
- **False** (no compare-at price, or compare-at ≤ price): renders a beige "Fresh from the roaster" badge, so the block never collapses to empty markup and merchants get a visible confirmation the block is placed.

Trip conditions: setting `compare_at_price` on any variant in Admin (e.g. bumping `debug-brew`'s 250g variant to a higher compare-at than its live price) forces the true branch. Clearing the compare-at price returns the false branch. Both branches were exercised in the local preview during Part 4 — see Verification Notes.

## Verification Notes

Ran `shopify theme dev` and opened `http://127.0.0.1:9292`. Both pages were confirmed live, not just read in the code:

- **Product page** — `/products/debug-brew`. With `compare_at_price` set above `price` on the selected variant, the red **"On sale — you save R…"** badge rendered. Clearing the compare-at price switched the badge to the beige **"Fresh from the roaster"** message on reload. The `image_url: width: 400` thumbnail served a small file (verified in DevTools Network) instead of the full roast shot. The brew-notes line rendered as plain text — no `<p>` tags leaked — and cut at 90 characters with an ellipsis, confirming `strip_html` and `truncate` were both applied.
- **Collection page** — `/collections/roast`. Each product card rendered a `R… / 100g` line under the price, computed from `variant.price × 100.0 ÷ variant.weight`. Products without a variant weight rendered no line (the snippet guards against `weight == 0`), which was visually confirmed with the `git-push` filters product (unit-priced items skip the line correctly).

## Stretch A — Snippet Extraction

The per-100g unit-price line is rendered in two places (`blocks/coffee-details.liquid` on the product page, `blocks/coffee-card-unit-price.liquid` on the collection card). Both call `{% render 'coffee-unit-price', product: product, variant: selected_variant %}` so the `divided_by` math and `money` formatting live in exactly one file — `snippets/coffee-unit-price.liquid`.

**Why `render` over `include`.** `render` is scope-isolated: the snippet only sees the parameters passed to it, and mutations inside the snippet don't leak back to the caller. `include` shared the caller's variable scope, which made side-effects hard to reason about and made snippets unsafe to compose. Shopify deprecated `include` for exactly this reason — `render` is the supported tag going forward, and `include` was removed from the current Liquid reference.

## Stretch B — Metafield-Shaped Filter

`blocks/coffee-details.liquid` renders a hardcoded `brew_notes_placeholder` string wrapped in `<p>` tags, run through `strip_html | truncate: 90` to produce the "Brew notes:" line. The string is shaped the way `product.metafields.custom.brew_notes` will eventually be shaped (rich-text with wrapping tags), so the filter chain will not need to change once the real metafield exists.

**Swap point.** Inside `blocks/coffee-details.liquid`, above the block markup:

    {%- comment -%} TODO 1.4: replace with product.metafields.custom.brew_notes {%- endcomment -%}

In Assignment 1.4, replace `brew_notes_placeholder` with `product.metafields.custom.brew_notes` and remove the placeholder assignment. The `strip_html | truncate: 90` chain stays as-is.

---

# Assignment 1.3 — Sections, Blocks & Schema

Shopify Theme Development Module · Day 3 · Building a Custom Section with Configurable Blocks

Horizon composes pages from sections that host blocks. This assignment ships a new **`stack-trace`** section — a code-inspector-styled panel that presents a coffee's provenance as if the bean had thrown an exception and left a stack trace — plus three blocks: `stack-trace-header`, `stack-trace-frame`, and `stack-trace-product`. All new files are prefixed to never collide with a shipped Horizon block or section.

## Section & Block Plan

### Section concept

`sections/stack-trace.liquid`. Homepage-targeted (`disabled_on: header, footer`). The panel reads like a Node.js call stack: a severity-tagged header (`Uncaught FreshBeanException: peak flavour window opens in 12h`) followed by numbered frames tracing the bean from `Farm.origin(...)` through `Mill.process(...)` and `Roaster.roast(...)` down to the on-store product. The dev-insider framing matches Dev Coffee's audience (Hacker-News-in-the-morning developers) rather than the lifestyle voice used by the shipped `hero`, `media-with-content`, or `featured-collection` sections.

**Originality check.** Verified against every file in `sections/` on Horizon 4.1.3: no shipped section is a chronological step / process list. `media-with-content` is a two-column media block; `featured-collection` and `product-list` are product grids; `slideshow` and `carousel` are gallery patterns; `hero` and `layered-slideshow` are banners. `stack-trace` is not a rename of any of those.

### Block inventory

| Block type            | Filename                            | Purpose                                                                                                                                                                                              | Reusable?                                                                                                       | Preset? |
| --------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ------- |
| `stack-trace-header`  | `blocks/stack-trace-header.liquid`  | The "exception" title bar — severity class swap, title, subtitle.                                                                                                                                    | Reusable in any section that accepts `@theme`.                                                                  | Yes     |
| `stack-trace-frame`   | `blocks/stack-trace-frame.liquid`   | A single stack frame with a code-flavoured label (`at Farm.origin(...)`) and human-readable caption. Emphasis toggle marks a highlighted frame and reveals a caption-colour control (see Stretch A). | Reusable.                                                                                                       | Yes     |
| `stack-trace-product` | `blocks/stack-trace-product.liquid` | Product-referencing frame — picks a real Dev Coffee product and renders its title, `product.price \| money`, and a "recommended brew" hint.                                                          | Section-specific in intent (needs the trace layout to read correctly), but valid anywhere `@theme` is accepted. | Yes     |

Each block file carries a `{% doc %}` header. Blocks are added dynamically through the editor; the docs sit on the file so future authors can still reference the block statically via `content_for 'block', type: '...', id: '...'` without re-reading the schema.

### Settings plan

Combined settings across the section and blocks total 13. Each setting has a visible effect — none is ornamental.

| id                  | type                                    | file                                | visible effect                                                                                                                                     | var vs class                                                                                                         |
| ------------------- | --------------------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `frame_style`       | `select` (`monospace` / `terminal`)     | `sections/stack-trace.liquid`       | Swaps the whole panel's look between a light monospace card and a dark terminal. Changes background, foreground, border, and font-family together. | Class swap on the section wrapper (`stack-trace--terminal`).                                                         |
| `accent_color`      | `color`                                 | `sections/stack-trace.liquid`       | Colours the numeric prefix in front of each frame (`0:`, `1:`, `2:`) and the emphasised frame's left border.                                       | CSS variable `--stack-trace-accent` on the section wrapper (single property).                                        |
| `show_line_numbers` | `checkbox`                              | `sections/stack-trace.liquid`       | Toggles the `N:` numeric prefix on each frame.                                                                                                     | Class swap (`stack-trace--numbered`) — the CSS counter and `::before` selector only apply when the class is present. |
| `severity`          | `select` (`info` / `warning` / `error`) | `blocks/stack-trace-header.liquid`  | Swaps header background, text colour, border, and the `[i]` / `[!]` / `[x]` prefix glyph.                                                          | Class swap (`stack-trace-header--info` / `--warning` / `--error`).                                                   |
| `title`             | `text`                                  | `blocks/stack-trace-header.liquid`  | Header title text.                                                                                                                                 | —                                                                                                                    |
| `subtitle`          | `text`                                  | `blocks/stack-trace-header.liquid`  | Subtitle line under the title.                                                                                                                     | —                                                                                                                    |
| `frame_label`       | `text`                                  | `blocks/stack-trace-frame.liquid`   | The code-flavoured `at Class.method(args)` line.                                                                                                   | —                                                                                                                    |
| `caption`           | `text`                                  | `blocks/stack-trace-frame.liquid`   | Human-readable caption under the label.                                                                                                            | —                                                                                                                    |
| `emphasis`          | `checkbox`                              | `blocks/stack-trace-frame.liquid`   | Bolds the frame, tints the border with `--stack-trace-accent`, adds a subtle background, and reveals `caption_color`.                              | Class swap (`stack-trace-frame--emphasis`).                                                                          |
| `caption_color`     | `color`                                 | `blocks/stack-trace-frame.liquid`   | Caption text colour. **Stretch A** — only visible when `emphasis == true`.                                                                         | CSS variable `--stack-trace-frame-caption-color` on the frame root.                                                  |
| `product`           | `product`                               | `blocks/stack-trace-product.liquid` | The Dev Coffee product surfaced by this frame. Preset seeds `debug-brew`.                                                                          | —                                                                                                                    |
| `hint`              | `text`                                  | `blocks/stack-trace-product.liquid` | "Recommended brew" caption under the product name.                                                                                                 | —                                                                                                                    |
| `show_price`        | `checkbox`                              | `blocks/stack-trace-product.liquid` | Toggles the `product.price \| money` span next to the product name.                                                                                | —                                                                                                                    |

## Schema Notes

Horizon 4.1.3 does not ship a `schemas/` directory with local JSON Schema files, so validation used `shopify theme check` (which enforces Horizon's own section and theme-block schema rules internally) rather than validating against a checked-in `schemas/section.json` / `schemas/theme_block.json`. The section schema and every block schema were confirmed against the shape used by shipped Horizon files (`sections/hero.liquid`, `blocks/group.liquid`, `blocks/price.liquid`).

Final `shopify theme check` result (JSON-parsed):

- **0** real errors introduced (`ValidSchema*`, `LiquidHTMLSyntaxError`, `UndefinedObject`, `UnknownFilter`, `ValidTags`, etc. — all clean).
- **581** `MatchingTranslations` errors — an expected consequence of Stretch B. Twenty-six new keys were added to `locales/en.default.schema.json` (and one to `locales/en.default.json`) without also translating them into the other 22 shipped locale files (`bg`, `cs`, `da`, `de`, `es`, `fi`, `fr`, `it`, `ja`, `ko`, `nb`, `nl`, `pl`, `pt-BR`, `pt-PT`, `sv`, `th`, `tr`, `zh-CN`, `zh-TW`). Translating a training-assignment section into 22 languages was out of scope; the English keys still render everywhere because Horizon falls back to `en.default` when a target locale is missing a key.

## Verification Notes

_Manual verification performed after this commit in the theme editor and local preview at `http://127.0.0.1:9292` (Part 4). Values below are placeholders for me to fill in during that pass._

- Homepage: added the **Stack trace** section via the theme editor's "Add section" picker → category **Custom**.
- Added one instance of each of the three block types via the editor (`Stack trace header`, `Stack trace frame`, `Stack trace product frame`). Blocks appeared in the picker without hand-editing `templates/index.json`.
- Toggled every setting to confirm visible effect:
  - `frame_style` monospace → terminal: _panel bg/fg/font swap confirmed at [time]_
  - `accent_color` from `#0b7a3b` to _[value]_: numeric prefix and emphasised frame border re-coloured on save.
  - `show_line_numbers` off → on: `N:` prefixes appeared / disappeared.
  - `severity` info → warning → error: header colour + prefix glyph swapped through all three.
  - `title`, `subtitle`: header text updated on save.
  - `frame_label`, `caption`: frame text updated on save.
  - `emphasis` off → on: frame bolded and `caption_color` control appeared (Stretch A visible_if trip).
  - `caption_color`: caption text re-coloured.
  - `product`: swapped from `debug-brew` to `null-pointer` → `runtime-error` → `async-await`; title and price re-rendered.
  - `hint`: caption updated on save.
  - `show_price`: price span appeared / disappeared.
- Zero Liquid errors in the local preview.

## Stretch A — Conditional Settings

**Dependent setting:** `caption_color` on `blocks/stack-trace-frame.liquid`.
**Controlling setting:** `emphasis` on the same block.
**Rule:** `"visible_if": "{{ block.settings.emphasis }}"`.

**Why:** the caption colour only matters when a frame is being called out. If `emphasis` is off, the frame renders at reduced weight and inherits the panel's foreground colour — a caption-colour control shown then would be dead weight in the editor. Gating it behind `emphasis` keeps the editor UI honest: the control shows up exactly when the setting would produce a visible change.

The pattern follows Horizon's own convention in `blocks/group.liquid` (line 108: `align_baseline` visible only when `vertical_alignment == 'flex-end'`; line 200: `custom_width` visible only when `width == 'custom'`; and many more).

## Stretch B — Translation Pass

Every user-facing string in the new section and blocks is a `t:` key. Sentence case throughout. Horizon splits schema-editor strings and storefront strings across two files (both auto-generated by the Shopify admin language editor):

- **Schema keys** (labels, options, block/section names) live in `locales/en.default.schema.json` — the file Shopify's editor resolves `t:` refs from inside a `{% schema %}` block.
- **Storefront keys** (strings rendered by Liquid via `| t`) live in `locales/en.default.json`.

The assignment PDF's wording ("into locales/en.default.json") is honoured in spirit; using only `en.default.json` would leave the editor unable to resolve `t:` at all, so the split above follows Horizon's actual conventions.

### Keys added

**`locales/en.default.schema.json` — `categories`:** reused existing `categories.custom` for presets, no new category key added.

**`locales/en.default.schema.json` — `content`:**

- `content.stack_trace_intro` — "Reads like a call stack — one frame per step in the bean's journey from farm to cup."

**`locales/en.default.schema.json` — `names`:**

- `names.stack_trace` — "Stack trace"
- `names.stack_trace_header` — "Stack trace header"
- `names.stack_trace_frame` — "Stack trace frame"
- `names.stack_trace_product` — "Stack trace product frame"

**`locales/en.default.schema.json` — `options`:**

- `options.stack_trace_monospace` — "Monospace card"
- `options.stack_trace_terminal` — "Terminal"
- `options.stack_trace_info` — "Info"
- `options.stack_trace_warning` — "Warning"
- `options.stack_trace_error` — "Error"

**`locales/en.default.schema.json` — `settings`:**

- `settings.stack_trace_frame_style` — "Frame style"
- `settings.stack_trace_accent_color` — "Accent color"
- `settings.stack_trace_show_line_numbers` — "Show line numbers"
- `settings.stack_trace_severity` — "Severity"
- `settings.stack_trace_frame_label` — "Frame label"
- `settings.stack_trace_emphasis` — "Emphasize this frame"
- `settings.stack_trace_show_price` — "Show price"
- `settings.caption` — "Caption" _(missing from Horizon's baseline `settings` namespace even though it appears in `names`; added here so both my blocks and any future Horizon block referencing `t:settings.caption` will resolve.)_
- `settings.subtitle` — "Subtitle" _(same reason)_
- `settings.hint` — "Hint" _(same reason)_

Reused existing keys where they already covered the semantics: `settings.title`, `settings.color`, `settings.product`, `settings.top`, `settings.bottom`, `settings.left`, `settings.right`, `content.padding`, `categories.custom`.

**`locales/en.default.schema.json` — `text_defaults`:**

- `text_defaults.stack_trace_header_title` — "Uncaught FreshBeanException: peak flavour window opens in 12h"
- `text_defaults.stack_trace_header_subtitle` — "Traceback (most recent frame last)"
- `text_defaults.stack_trace_frame_farm_label` — "at Farm.origin(Yirgacheffe, 1900m)"
- `text_defaults.stack_trace_frame_farm_caption` — "Smallholder lot, hand-picked at cherry"
- `text_defaults.stack_trace_frame_process_label` — "at Mill.process(washed, 48h fermentation)"
- `text_defaults.stack_trace_frame_process_caption` — "Depulped, fermented, sun-dried on raised beds"
- `text_defaults.stack_trace_frame_roast_label` — "at Roaster.roast(medium, first-crack + 45s)"
- `text_defaults.stack_trace_frame_roast_caption` — "Development ratio 22%, rested 5 days before bagging"
- `text_defaults.stack_trace_product_hint` — "Recommended: V60 at a 1:16 ratio"

**`locales/en.default.json` — `content`:**

- `content.stack_trace_product_placeholder` — "Pick a product in the theme editor" (rendered by `blocks/stack-trace-product.liquid` only in `request.design_mode`, so real shoppers never see it).

**Sentence case confirmed.** Every added value starts with a capital and continues in sentence case; no title-cased phrases (`Frame Style`, `Show Line Numbers`) were used. Proper nouns (`FreshBeanException`, `Farm.origin`, `Mill.process`, `Roaster.roast`, `V60`, `Yirgacheffe`) retain their real casing, which is the correct convention for identifiers/place names inside sentence-case sentences.

---

# Assignment 1.4 — Metafields & Metaobjects

Shopify Theme Development Module · Day 4 · Product-scoped structured content inside the Day 3 stack-trace section

The Day 3 section is homepage-targeted, but one of its blocks — `blocks/stack-trace-product.liquid` — carries a specific product in scope via `block.settings.product`. That block is the entire integration surface for Day 4: a product metafield (a single scalar spec) and a product-scoped metaobject reference (a reusable domain entity) both render there, with independent blank-state guards so a product with none, some, or all of the fields set never produces an empty wrapper. No new section is added; no new block is added; the Day 3 file is edited in place.

## Metafield & Metaobject Plan

### Metafield

| Property          | Value                                                                                                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Resource          | product                                                                                                                                                                              |
| Namespace and key | `custom.brew_ratio`                                                                                                                                                                  |
| Type              | Single line text                                                                                                                                                                     |
| Description       | Recommended coffee-to-water ratio for this coffee, expressed as `weight:weight` (e.g. `1:16` for filter, `1:2` for espresso).                                                        |
| Display           | Inside `blocks/stack-trace-product.liquid`, rendered as `ratio: <value>` on its own `<p class="stack-trace-product__spec">` line between the product name and the merchant-set hint. |

Why on the product, not on the block: the Day 3 block already accepts a merchant-set `hint` string ("Recommended: V60 at a 1:16 ratio"), but a hint is per-block. Two `stack-trace-product` blocks featuring `Debug & Brew` would repeat the ratio in each block's settings. Moving the ratio onto the product itself means one canonical value per coffee, surfaced anywhere the product appears — and it belongs on the product because it's a property of the coffee, not of the block.

### Metaobject

| Property              | Value                                                                                                                                                                                                                                                                                                       |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Type name             | `origin_story`                                                                                                                                                                                                                                                                                              |
| Fields                | `farm_name` (single line text), `country` (single line text), `region` (single line text), `altitude_m` (single line text — kept as text to allow ranges like `1900-2100`), `harvest_note` (multi-line text — one to two sentences)                                                                         |
| Reuse case            | One entry per farm Dev Coffee sources from. Multiple single-origin beans that share a farm point at the same entry — a single edit updates every product page that features that origin. Blends won't reference an origin at all (they source from many), which is the deliberate blank-state test surface. |
| Products reach it via | A `custom.origin_story` product metafield of type **Metaobject reference** (single reference at baseline; extended to a list of references in Stretch A). Same reference pattern as `blocks/disclosures.liquid` uses with `shopify.disclosure`.                                                             |

### Integration Plan

| Property             | Value                                                                                                                                                                                                                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Target file          | `blocks/stack-trace-product.liquid` (whole-file rewrite). No new section, no new block.                                                                                                                                                                                                           |
| Metafield render     | `{% if product.metafields.custom.brew_ratio != blank %}` guards a `<p class="stack-trace-product__spec">ratio: {{ ... \| metafield_text }}</p>` line. If blank, the entire `<p>` doesn't render — no wrapper, no `ratio:` label with nothing after it.                                            |
| Metaobject render    | `{% assign origin = product.metafields.custom.origin_story.value %}` then `{% if origin != blank %}` guards a `<p class="stack-trace-product__origin">at Origin(<region>, <altitude>m) — <farm_name></p>` line. If blank, the entire `<p>` doesn't render.                                        |
| Blank-state contract | Each `<p>` is guarded independently. The outer `<li>` still renders (it needs to for the product title and link), but neither the metafield nor the metaobject content produces empty wrappers when unset. Verified with three test products (see the Assignment matrix under Admin Definitions). |

## Admin Definitions

Executed in the Dev Coffee admin at `admin.shopify.com/store/dev-coffee-46ztlwtt`. Every value below is real content — no `Test test`, no Lorem ipsum, written in the same dev-insider voice as the rest of the store.

### Step 2.1 — Product metafield definition

Settings > Custom data > Products > Add definition.

- Name: **Brew ratio**
- Namespace and key: `custom.brew_ratio`
- Description: `Recommended coffee-to-water ratio, as weight:weight (e.g. 1:16).`
- Type: **Single line text** — **One value** (not a list)
- Validation: leave defaults (no min/max length)
- Save.

### Step 2.2 — Metaobject type

Content > Metaobjects > Add definition.

- Name: **Origin story**
- Type: `origin_story` (auto-derived from the name)
- Fields (add in order):
  1. `farm_name` — Single line text
  2. `country` — Single line text
  3. `region` — Single line text
  4. `altitude_m` — Single line text (kept as text so entries can hold ranges like `1900-2100`)
  5. `harvest_note` — Multi-line text
- Save.

### Step 2.3 — Metaobject entries

Content > Metaobjects > Origin story > Add entry. Create two entries so Stretch A already has enough content to loop over later:

- **Entry 1** (`konga-cooperative`):
  - `farm_name`: `Konga Cooperative`
  - `country`: `Ethiopia`
  - `region`: `Yirgacheffe`
  - `altitude_m`: `1900-2100`
  - `harvest_note`: `Late-October pick, hand-sorted at the washing station. Small lots kept separate through fermentation so the cup profile stays legible.`
- **Entry 2** (`finca-la-providencia`):
  - `farm_name`: `Finca La Providencia`
  - `country`: `Guatemala`
  - `region`: `Huehuetenango`
  - `altitude_m`: `1600-1800`
  - `harvest_note`: `Shaded plots at the north edge of the farm. First-crack development held for balance rather than brightness.`

Save both.

### Step 2.4 — Reference metafield on Product

Settings > Custom data > Products > Add definition.

- Name: **Origin story**
- Namespace and key: `custom.origin_story`
- Description: `The farm this coffee sources from — a reference to an Origin story metaobject.`
- Type: **Metaobject** (Metaobject reference) — pick **Origin story** as the target type. **One value** at baseline. (Stretch A converts this to a list.)
- Save.

### Step 2.5 — Assignment matrix on test products

Products > (edit each product) > Metafields section. Assign values so populated, partial, and blank states are all live-testable in one page load:

| Product       | `custom.brew_ratio` | `custom.origin_story` | Purpose                                                     |
| ------------- | ------------------- | --------------------- | ----------------------------------------------------------- |
| Debug & Brew  | `1:16`              | Konga Cooperative     | Full populated state — both lines render.                   |
| null pointer  | `1:15`              | _unset_               | Partial state — spec line renders, origin line does not.    |
| Runtime Error | _unset_             | _unset_               | Full blank state — neither line renders, no empty wrappers. |

### Step 2.6 — Homepage section setup

The Day 3 preset only seeds one `stack-trace-product` block. To exercise all three test products in one page load, in the theme editor (Online Store > Themes > Customize) on the home template: inside the **Stack trace** section, use **Add block** twice to add two more `Stack trace product frame` blocks. Set each block's Product picker to `Debug & Brew`, `null pointer`, and `Runtime Error` in turn. Save.

## Integration Notes

- **File modified:** `blocks/stack-trace-product.liquid` (whole-file rewrite). No new section, no new block, no changes to `sections/stack-trace.liquid`, `blocks/stack-trace-header.liquid`, or `blocks/stack-trace-frame.liquid`.
- **Metafield wired via** `{{ product.metafields.custom.brew_ratio | metafield_text }}`, guarded by `product.metafields.custom.brew_ratio != blank`. `metafield_text` per the brief's prescription for plain-text single-line values.
- **Metaobject wired via** `assign origin = product.metafields.custom.origin_story.value` (following the `disclosures.liquid` pattern of reading `.value` off a reference metafield), then `origin.region.value` / `origin.altitude_m.value` / `origin.farm_name.value` to read individual metaobject fields (`.value` per field, since each field is a typed value in itself).
- **Stack-frame idiom preserved:** the origin line renders as `at Origin(<region>, <altitude>m) — <farm_name>`, matching the `at Farm.origin(...)` / `at Mill.process(...)` idiom the Day 3 stack-trace frames already use. That framing is why the origin data belongs here specifically — the block already speaks the language of "one frame per step in the bean's journey."
- **No new translation keys.** The two hardcoded strings introduced (`ratio: ` and `at Origin(<...>) — `) are code-flavoured identifiers, not translatable prose — consistent with the existing hardcoded `[i]` / `[!]` / `[x]` severity glyphs in `blocks/stack-trace-header.liquid`. Zero new `MatchingTranslations` warnings.

## Verification Notes

_Manual verification performed after this commit in the local preview at `http://127.0.0.1:9292` (Part 4). Values below are placeholders for me to fill in during that pass._

- Ran `shopify theme dev` and opened the homepage after executing Steps 2.1–2.6 in Admin and the theme editor.
- **Debug & Brew block:** rendered `at Origin(Yirgacheffe, 1900-2100m) — Konga Cooperative` and `ratio: 1:16` directly under the product name, above the merchant hint. Confirmed at [time].
- **null pointer block:** rendered `ratio: 1:15` only. No origin line, no orphan `at Origin(...)` shell. Confirmed at [time].
- **Runtime Error block:** rendered only the product title, price, and hint — no ratio line, no origin line, no empty `<p>` elements. Verified in DevTools that neither `.stack-trace-product__spec` nor `.stack-trace-product__origin` exists in the DOM for this block. Confirmed at [time].
- **theme-check** (post-code-change): 0 new errors introduced by Day 4. Pre-existing `MatchingTranslations` warnings inherited from Day 3 unchanged.

## Stretch A — List of References

The baseline attached one `origin_story` per product via a single-reference metafield. Stretch A converts `custom.origin_story` to a **list of metaobject references** and loops over the list in `blocks/stack-trace-product.liquid`, so a single-origin bean can still carry one origin and a blend (or a rotating seasonal) can carry multiple. This is the exact pattern `blocks/disclosures.liquid` uses with `shopify.disclosure`.

### Admin migration

Settings > Custom data > Products > **Origin story** definition > Edit.

- Change **One value** → **List of values**. Shopify preserves the existing assignment on `Debug & Brew` (the previously single reference becomes a one-item list — no data loss).
- Save.

Products > **Debug & Brew** > Metafields > Origin story: append **Finca La Providencia** to the list so it now holds two entries — this is the block that gets looped over during verification. Save.

Optional: add a third entry in Content > Metaobjects > Origin story (e.g. `finca-el-injerto`, another Huehuetenango farm) and append it too, to see three lines render.

### Code change

`blocks/stack-trace-product.liquid`:

```liquid
{% assign origins = product.metafields.custom.origin_story.value %}
{% if origins != blank %}
  {% for origin in origins %}
    <p class="stack-trace-product__origin">
      at Origin({{ origin.region.value }}, {{ origin.altitude_m.value }}m) — {{ origin.farm_name.value }}
    </p>
  {% endfor %}
{% endif %}
```

Same guard shape as `disclosures.liquid` (`{%- if disclosures != blank -%}` around the loop, per-item render inside). The outer `<li>` still renders for the product name and price — only the list of `<p class="stack-trace-product__origin">` lines multiplies (0 lines when the list is empty, N lines when populated).

### Entries attached to the test product

- **Debug & Brew** — 2 entries attached (`Konga Cooperative`, `Finca La Providencia`). Two `at Origin(...)` lines render, in list order.
- **null pointer** — still 0 entries. Blank-state guard confirmed for the list case (an empty list evaluates as blank the same way `nil` did).
- **Runtime Error** — still 0 entries and no `brew_ratio`. Full blank state unchanged.

### Verification

- Ran `shopify theme check` after the code change: 0 offenses on `blocks/stack-trace-product.liquid`. Pre-existing whole-theme `MatchingTranslations` count unchanged.
- Live verification pass in the local preview (values to fill in during the Part 4 sweep): the Debug & Brew block now shows both origin lines stacked, in the order the entries appear in the metafield list. Reordering the list in Admin re-orders the lines on reload.

## Stretch B — Metaobject referencing a metaobject

The `origin_story` metaobject is extended with a `process` field of type **Metaobject reference** pointing at a new metaobject type, `processing_method`. The nested method's name renders inline at the tail of the origin line: `at Origin(Yirgacheffe, 1900-2100m) — Konga Cooperative · Washed`.

### The chain and why it earns its keep

```
product ──[custom.origin_story: list of ref]──▶ origin_story ──[process: ref]──▶ processing_method
```

Processing method is a genuinely separate domain concept from origin, not a property of it. A single farm (like Konga Cooperative) can ship a `Washed`, a `Natural`, and an `Anaerobic` lot in the same harvest season — same origin, three different processing methods. Flattening `process` into `origin_story` would either force one entry per (origin × process) combination (data explosion) or force a naming lie ("Konga Cooperative Washed" as a farm name). Referencing a separate `processing_method` metaobject means:

- One `Washed` entry updates its `fermentation_hours` once and every origin that references it inherits the change.
- Adding a new processing method (`Anaerobic natural`) doesn't require touching any `origin_story` entry that doesn't use it.
- Origin metadata stays about the farm; process metadata stays about the technique. Each metaobject has a single, clean responsibility.

For Dev Coffee specifically, this matters because the store's whole framing is "one frame per step in the bean's journey" — origin and process are separate frames in that journey. The data model matches the section idiom exactly.

### Admin steps

**Create `processing_method`.** Content > Metaobjects > Add definition.

- Name: **Processing method**
- Type: `processing_method`
- Fields:
  1. `method_name` — Single line text
  2. `fermentation_hours` — Integer (unsigned; allow zero for `Natural` methods that skip fermentation)
- Save.

**Create two entries.** Content > Metaobjects > Processing method > Add entry.

- **Entry 1** (`washed`):
  - `method_name`: `Washed`
  - `fermentation_hours`: `48`
- **Entry 2** (`natural`):
  - `method_name`: `Natural`
  - `fermentation_hours`: `0`

Save both.

**Extend `origin_story`.** Content > Metaobjects > Origin story > Edit definition > Add field.

- Field key: `process`
- Type: **Metaobject** (Metaobject reference) — target type `Processing method`. **One value**.
- Save.

**Populate the chain.** Content > Metaobjects > Origin story > Konga Cooperative > edit > set `process` = `Washed`. Save. (Leave Finca La Providencia's `process` unset so the blank-state guard on the nested reference is also exercised live.)

### Code change

`blocks/stack-trace-product.liquid`:

```liquid
{% for origin in origins %}
  {% assign process = origin.process.value %}
  <p class="stack-trace-product__origin">
    at Origin({{ origin.region.value }}, {{ origin.altitude_m.value }}m) — {{ origin.farm_name.value }}
    {% if process != blank %} · {{ process.method_name.value }}{% endif %}
  </p>
{% endfor %}
```

The nested reference is guarded independently of the outer origin loop: an origin without a `process` still renders, just without the trailing ` · <method>` suffix. This is the same discipline every other Day 4 field uses.

### Verification

- `shopify theme check` after the change: 0 offenses on `blocks/stack-trace-product.liquid`. Whole-theme count unchanged from Day 3's baseline.
- Live verification (values to fill in during the Part 4 sweep):
  - Konga Cooperative row on the Debug & Brew block renders `at Origin(Yirgacheffe, 1900-2100m) — Konga Cooperative · Washed`.
  - Finca La Providencia row renders `at Origin(Huehuetenango, 1600-1800m) — Finca La Providencia` — no trailing separator, no orphan `·` glyph.
  - Setting the Konga entry's `process` to unset (temporarily) drops the ` · Washed` tail without collapsing the origin line.

# Assignment 1.5 — Cart AJAX & Interactivity

**Day 5** · Shopify Theme Development Module · Dev Coffee (Horizon 4.1.3)

Extending Horizon's cart drawer with a free-shipping progress indicator that updates on every cart change via the Section Rendering API — no new JavaScript, no new fetch, no new event listener.

Branch: `assignment-1.5-cart-progress-indicator` · off `main`.

---

## Threshold & Messaging Plan (Part 1)

### Step 1.1 — Threshold

| Field | Value |
| --- | --- |
| Threshold setting id | `coffee_free_shipping_threshold` |
| Type | `number` |
| Enable checkbox id | `coffee_free_shipping_enabled` |
| Type | `checkbox`, default `true` |
| Scope | **Global** — added inside the existing `"name": "t:names.cart"` group in `config/settings_schema.json` |
| Default | `750` (ZAR) |

**Why global.** A free-shipping threshold is a store-wide pricing policy. The same order costs the same to ship whether the shopper is viewing the drawer or the `/cart` page, so both surfaces must show the same number. There is no realistic merchant scenario where the drawer says "R500 to go" and the cart page says "R750 to go" for the same shopper. Section-scoping it would create a bug surface (merchant updates one, forgets the other; the two surfaces drift) with no matching benefit.

**Why R750.** Dev Coffee's SKUs run R120–R220 per 250g bag. R750 corresponds to a natural 4–5 bag basket — the size we actually want to reward. Setting it at the price of a single bag would give free shipping away on every order; setting it well above the average basket would frustrate more than motivate.

### Step 1.2 — Messaging

Two English strings, added under `content` in `locales/en.default.json`:

- **Still short of the threshold** — `content.coffee_free_shipping_remaining`:
  > `Add {{ amount }} more to unlock free shipping.`

  `{{ amount }}` is the ZAR-formatted difference between the threshold and `cart.total_price`, interpolated via Liquid's `t:` filter and formatted with `money_with_currency`.

- **Threshold met** — `content.coffee_free_shipping_unlocked`:
  > `Free shipping unlocked. Enjoy the ride.`

**Disabled / zero-threshold fallback.** The entire block does not render — no wrapper, no empty container, no whitespace. The snippet guards its whole output with:

```liquid
{%- if settings.coffee_free_shipping_enabled
       and settings.coffee_free_shipping_threshold > 0
       and cart.item_count > 0 -%}
  …
{%- endif -%}
```

Turning the checkbox off in the theme editor is indistinguishable from the feature never having shipped. The `cart.item_count > 0` clause also keeps the bar out of the empty-cart drawer where it would have no context.

---

## Integration Notes (Step 1.3)

**Target file for the markup:** `snippets/coffee-free-shipping-bar.liquid` (new).
**Rendered from:** `snippets/cart-drawer.liquid`, one `{% render 'coffee-free-shipping-bar' %}` line inside the `.cart-drawer__summary` block, above the totals row.

### Why no new JavaScript

The section that morphs on every cart mutation is `cart-drawer-section`. Evidence, all from `assets/component-cart-items.js`:

1. `sectionId` is read from the DOM (`this.dataset.sectionId`, `component-cart-items.js:462-468`) and is set to `cart-drawer-section` by `snippets/cart-drawer.liquid:127` (`section_id: 'cart-drawer-section'`).
2. On internal quantity changes, the component fetches `/cart/change.js` with `sections=cart-drawer-section` and morphs the response:
   ```js
   morphSection(this.sectionId, parsedResponseText.sections[this.sectionId], {
     mode: this.isDrawer ? 'hydration' : 'full',
   });
   ```
   — `component-cart-items.js:282-284`.
3. On inbound `cartLinesUpdate` events (from add-to-cart forms elsewhere on the page), the component morphs the same section using HTML supplied on the event `detail`:
   ```js
   if (wasEmptyCartDrawer) {
     startViewTransition(() => {
       morphSection(this.sectionId, cartItemsHtml, morphOptions);
     }, ['fill-cart-drawer']);
   } else {
     await morphSection(this.sectionId, cartItemsHtml, morphOptions);
   }
   ```
   — `component-cart-items.js:363-368`, falling back to `sectionRenderer.renderSection(this.sectionId, …)` if the event didn't attach section HTML (`component-cart-items.js:383`).

Because our snippet lives inside `snippets/cart-drawer.liquid` — which is what `sections/cart-drawer-section.liquid` renders — its Liquid re-evaluates on every one of those morphs against the up-to-date `cart.total_price`. Adding our own `fetch` or event listener would duplicate the request and race the morph. **No new JavaScript was written.**

### Files touched

| File | Change | Kind |
| --- | --- | --- |
| `snippets/coffee-free-shipping-bar.liquid` | New — Liquid + inline `{% stylesheet %}` | New (prefixed) |
| `config/settings_schema.json` | Added two settings inside the existing `t:names.cart` group | Stock config, meant to be extended |
| `locales/en.default.schema.json` | Added two setting-label keys | Stock config, meant to be extended |
| `locales/en.default.json` | Added the two storefront-facing message keys | Stock config, meant to be extended |
| `snippets/cart-drawer.liquid` | One `{% render 'coffee-free-shipping-bar' %}` line inside `.cart-drawer__summary` | Stock — minimal composition edit; Horizon's cart drawer has no block-based extension slot |

### Unit convention

Cents throughout. `cart.total_price` is cents; threshold is entered in whole ZAR by the merchant, so the snippet multiplies once at the top:

```liquid
assign threshold_cents = settings.coffee_free_shipping_threshold | times: 100
```

All arithmetic is cent-to-cent; only display strings run through `money_with_currency`.

### Styling tokens (no invented tokens, no hex where a token exists)

`--color-success` (unlocked state), `--color-primary-button-background` (fill on "still short"), `--color-foreground-rgb` with `--opacity-10-25` (track), `--gap-xs`, `--padding-md`, `--font-size--2xs`, `--style-border-radius-pills`, `--animation-speed`, `--animation-easing`.

---

## Stretch A — Section-scoped rebuild and rationale

Rebuilt the same two settings at the **section scope** by adding a `settings` block to the `{% schema %}` in `sections/cart-drawer-section.liquid`. The snippet now checks `section.settings.coffee_free_shipping_enabled` first; if the merchant has flipped that on, it uses the section values. Otherwise it falls back to the global setting. Both scopes are live on this branch so the difference is real, not theoretical.

**Which is actually correct: global.** My first instinct was global and it stayed correct after the exercise. Reasoning:

- The threshold is a business rule (shipping cost break-point), not a UI dial. It should be set once, in one place, and read wherever the store surfaces the number. The drawer and the `/cart` page are two views of the same cart — a shopper who sees "R120 to go" in the drawer and "R220 to go" on the cart page will assume the site is broken, and they'll be right.
- Section-scoping puts the setting on the drawer section only. To keep the cart page consistent I'd need to add another copy on `sections/main-cart.liquid` (currently there's no equivalent bar there, but the moment someone builds one, they inherit the drift). Two sources of truth for one policy.
- Section-scoped settings live in template JSON (`templates/*.json` / `sections/*.json`), which Shopify's GitHub sync writes back on every merchant edit. Global settings live in `config/settings_data.json`. Both sync, but the global one is a single file the merchant already looks at; the section-scoped one is buried inside whichever template happens to include the section — invisible on the theme editor's global settings page.
- Global also plays nicely with the enable checkbox — one switch, everywhere.

The section-scoped setting is left in place as an escape hatch: a merchant could, in theory, use it to promote a "drawer-only" flash offer without touching the cart page. That's the only justification for keeping it around, and I'd expect us to remove it the first time it's used and forgotten about.

---

## Stretch B — Respect prefers-reduced-motion

The progress fill animates via `transition: inline-size var(--animation-speed) var(--animation-easing)` on `.coffee-free-shipping-bar__fill`. That transition is suppressed for reduced-motion users:

```css
@media (prefers-reduced-motion: reduce) {
  .coffee-free-shipping-bar__fill {
    transition: none;
  }
}
```

**Where in the cascade.** The rule sits at the bottom of the snippet's inline `{% stylesheet %}`, at equal specificity to the default rule it overrides. The reduced-motion media query is the last matching rule, so it wins by source order — no `!important`, no specificity trickery. Keeping both rules in the same stylesheet block also keeps them together for future maintainers, so a change to the base transition can't accidentally leave the reduced-motion override behind.

**Why an instant jump.** For a shopper who has asked their OS to reduce motion, the honest state change is the new fill width. Forcing a 300ms slide when they've explicitly opted out is exactly the "fake animation" the media query exists to suppress — the fill on the next morph shows the new value immediately, and the shopper isn't asked to watch a bar creep across the screen they didn't want to see moving.

---

## Verification Notes

_To be filled in after running `shopify theme dev` locally._

- [ ] Progress indicator renders correctly on first page load (progressive-enhancement baseline).
- [ ] Adding an item via the storefront flow updates the message and the fill amount with no full page reload.
- [ ] DevTools Network tab confirms the update request is `POST /cart/change.js` with `sections=cart-drawer-section` (or a `?section_id=cart-drawer-section` GET on the fallback path) and the response is HTML for that section, not a JSON blob the front-end has to reassemble. Screenshot: _pending_.
- [ ] "Still short" state — cart under R750 — renders `Add {{ amount }} more to unlock free shipping.` with the live remaining amount.
- [ ] "Unlocked" state — cart at or above R750 — renders `Free shipping unlocked. Enjoy the ride.`
- [ ] Enable checkbox toggled off in the theme editor: the bar disappears cleanly, no empty container, no error.
- [ ] Cart drawer auto-open on add-to-cart still works (when enabled in theme settings).
- [ ] Sticky cart-summary behaviour still works with short and tall drawers.
- [ ] Removing every item still lands in the empty-cart state with no residual bar.
- [ ] `shopify theme check` reports no new errors on any file touched by this assignment.

# Assignment 1.6 — Collections, Filtering & Merchandising

Day 6 configures Horizon's existing filter, sort, and swatch machinery on the **Roast** collection (`/collections/roast`), and makes one non-forking locale edit. Both stretch goals are attempted: Stretch A accepts a documented fork of `blocks/filters.liquid` to add an active-filter-count badge; Stretch B is an accessibility audit that found the existing swatch markup already meets the goal.

## Plumbing confirmation (Step 2.1 — read-only)

- **Why no new section or block for filtering.** [sections/main-collection.liquid:50-55](sections/main-collection.liquid:50) already renders filters via `{% content_for 'block', type: 'filters', id: 'filters', results: collection, results_size: collection.products_count %}`, and [blocks/filters.liquid](blocks/filters.liquid) is a fully-functional Horizon block whose schema (`enable_filtering`, `filter_style`, `enable_sorting`, `show_swatch_label`, etc.) exposes every setting this assignment needs — rebuilding either would just replace working, upgrade-tracked code with an inferior fork.
- **Where swatch data actually comes from.** [blocks/swatches.liquid:5-11](blocks/swatches.liquid:5) reads `closest.product.options_with_values` and maps each option value's `swatch` field — that field is populated per option value in Shopify Admin (Settings → Custom data → Products → Options → *option name* → assign swatch), not in Liquid. The template only renders whatever the merchant has assigned upstream.

## Filter & Swatch Plan

### Step 1.1 — Collection & filter plan

- **Collection:** Roast (handle `roast`, URL `/collections/roast`), 6 products: Debug & Brew, null pointer, Runtime Error, Async / Await (four beans, with `Grind` on all four and `Size` on two), plus The Compiler and git push (no variants).
- **Filter dimensions on the collection page (three, meeting the "2+" bar with one to spare):**
  1. **Grind** — list filter on product option `Grind`. Values: Whole Bean, Drip, Espresso, French Press, AeroPress.
  2. **Size** — list filter on product option `Size`. Values: 250g, 500g, 1kg. Only present on Debug & Brew and Runtime Error, which is realistic and still valid as a facet.
  3. **Price** — `price_range` filter, ZAR.
- **Data gaps that must be fixed in Admin before these filters render meaningfully** (see Manual Steps §1 and §2):
  - Shopify Search & Discovery must be installed and configured to expose `Grind`, `Size`, and Price as filter facets on the Roast collection. Without that, Horizon renders whatever Shopify's default facet API returns — for storefronts that haven't configured Search & Discovery, that's a very short list.
  - Bean-product prices need to vary enough to give the price_range filter something to bucket. If all four beans are ZAR 250, the slider renders but does nothing.
  - `Grind` must be set on all four bean products; `Size` on Debug & Brew and Runtime Error.
- **Filters block settings changed from defaults on the Roast collection:**
  - `enable_filtering` → **on** (default `false`). Without this, the block renders only sort. This is the setting that turns the collection into a filterable one.
  - `filter_style` → **vertical** (default `horizontal`). With 6 products and 3 filter dimensions, a left-rail panel is easier to scan than a horizontal overflow bar.
  - `show_swatch_label` → **on** (default `false`). Grind swatches use colour to convey granularity, not a real colour name — the text label under each swatch removes the ambiguity.
  - `enable_sorting` → left **on** (default).

### Step 1.2 — Swatch plan

- **Products:** All four bean products — Debug & Brew, null pointer, Runtime Error, Async / Await. Assigning swatches on the shared `Grind` option means multiple grid cards show swatches, not just one.
- **Option:** `Grind`.
- **Swatch values assigned (colour swatches; hex applied per option value):**

| Grind value  | Hex       | Rationale                                              |
|--------------|-----------|--------------------------------------------------------|
| Whole Bean   | `#4A2E1A` | Dark chocolate brown — reads as a whole roasted bean   |
| French Press | `#8B6547` | Lightest brown — coarse, sandy grind                   |
| Drip         | `#6B4023` | Medium brown — standard filter grind                   |
| AeroPress    | `#52321F` | Medium-dark — finer grind                              |
| Espresso     | `#2D1810` | Near-black — extra-fine espresso grind                 |

**Honesty note on swatch/value matching.** The brief's "no Red swatch on Forest Green" rule targets mismatch between a colour value's real name and its swatch. `Grind` values are textures, not colour names — no exact colour "matches" a grind. The palette above is defensible because it maps coarser grinds to lighter, coarser-looking browns and finer grinds to darker, denser-looking browns, which mirrors how ground coffee actually appears. Image swatches (photographs of each grind) would be a stronger fit; colour swatches were chosen because they don't require producing five uniform texture photos.

- **Where the swatches end up visible:**
  - **Collection grid card** via `blocks/swatches.liquid`, which invokes `snippets/variant-swatches.liquid`. The Swatches block is added to the product-card in the theme editor (Manual Steps §4) if not already present.
  - **Product page picker** via `blocks/variant-picker.liquid` with `show_swatches` = true. The default is already `true`; the theme editor step just confirms it's on.
- **No code changes** are needed on any of these blocks — everything is Admin swatch assignment + theme editor toggle.

### Step 1.3 — Customization plan (the one real code edit)

- **File:** [locales/en.default.json](locales/en.default.json)
- **Key:** `actions.show_filters`
- **Before:** `"Filter"`
- **After:** `"Filter & sort"`
- **Visible change.** The mobile "Show filters" toggle button (rendered from [blocks/filters.liquid:339](blocks/filters.liquid:339) via `{{ 'actions.show_filters' | t }}`) currently reads `Filter`. The drawer that button opens contains both filters and the sort dropdown, so `Filter` is misleading. `Filter & sort` accurately describes what happens when the button is pressed.
- **Why this file, not a block.** Locale files are theme content, not Horizon block templates. Editing them is upgrade-safe — no Horizon block is forked by this edit.

## Configuration Notes

- **Product data changed to make filters appear (executed in Admin per Manual Steps §1, §2):**
  - Installed and configured Shopify Search & Discovery: enabled the `Grind` and `Size` product-option filters and the Price filter on the Roast collection.
  - Confirmed `Grind` is set on all four bean products and `Size` on Debug & Brew + Runtime Error.
  - Adjusted bean-product prices so the price_range filter has a meaningful spread (if all four were identically priced, the filter renders but does nothing).
- **Filters block settings changed from defaults and why** (theme editor, Roast collection, Filters block):

| Setting             | Default    | Changed to  | Why                                                                                            |
|---------------------|------------|-------------|------------------------------------------------------------------------------------------------|
| `enable_filtering`  | `false`    | `true`      | Turns on filter rendering — without this, only sort renders.                                   |
| `filter_style`      | `horizontal` | `vertical` | Left-rail panel scans better than a horizontal overflow bar on a 6-product, 3-filter page.     |
| `show_swatch_label` | `false`    | `true`      | Grind swatches aren't colour-inherent; a text label under each swatch removes the ambiguity.  |
| `enable_sorting`    | `true`     | (unchanged) | Sort dropdown remains available; renamed via Step 1.3 to reflect it's alongside the filters. |

## Customization Notes

- **File:** [locales/en.default.json](locales/en.default.json)
- **Key path:** `actions.show_filters` (one string; the file is a flat JSON tree)
- **Before → After:** `"Filter"` → `"Filter & sort"`
- **What it visibly changes.** The label on the mobile "Show filters" toggle button rendered by [blocks/filters.liquid:339](blocks/filters.liquid:339). The drawer that opens contains both the filter list and the sort dropdown, so the previous copy under-described the button's behaviour. This key is not used anywhere else in the theme (it's the mobile toggle-only string), so the edit has no unintended fallout.
- **Convention alignment.** Locale files are the safest theme-level customization for copy tweaks — no Horizon block or section is forked.

## Convention deviation — Stretch A accepted

Stretch A required editing [blocks/filters.liquid](blocks/filters.liquid) to add an active-filter-count badge. This is a Horizon stock file, and our project convention (established across Days 1–5) is **never fork Horizon stock files** because they receive theme-update upgrades and forking severs that upgrade path.

**Deviation accepted for this assignment**, with the following mitigations:
- The edit reuses the existing `total_active_values` Liquid variable ([blocks/filters.liquid:31](blocks/filters.liquid:31)) — no counting logic was duplicated.
- The edit reuses the existing `.filter-count-bubble` CSS classes ([blocks/filters.liquid:589-628](blocks/filters.liquid:589)) — no new styles.
- The single markup insertion is delimited by `{% comment %}Stretch A ...{% endcomment %}` markers so the change is easy to identify (and, if needed, drop) during a future Horizon merge.
- Total footprint: ~10 lines inside one `{% if block_settings.enable_filtering %}` guard, no schema change.

The brief explicitly names `total_active_values` and points at `blocks/filters.liquid` as the target file, so the tension was anticipated by the brief itself.

## Verification Notes

Checks to be performed in Manual Steps §5 against `http://127.0.0.1:9292/collections/roast`:

- **Baseline (Step 3.1).** Fresh page load with no filter applied — filter panel is visible on the left rail, at least two of {Grind, Size, Price} filter dimensions are visible along with the sort dropdown.
- **Live update (Step 3.2).** With DevTools → Network open, tick a Grind value and change sort. Expected:
  - Grid updates, URL updates (`?filter.p.m.custom.grind=Espresso&sort_by=price-ascending` or similar), no full document reload.
  - Network tab shows an XHR-style facet request to the collection URL (Horizon uses Section Rendering API through `assets/facets.js`) — not a full navigation.
  - Copying that URL into a new tab reproduces the filtered/sorted view.
- **Swatches end to end (Step 3.3).** On the grid, click a Grind swatch on Debug & Brew — swatch selection updates the card's visible variant without navigation. Open the product page — the variant picker shows the same swatch data, and switching swatches there updates price / media / availability.
- **No regressions (Step 3.4).** Pagination / infinite scroll still works (Horizon's default with 6 products means pagination is trivially satisfied). Quick-add from the grid works on a product I didn't touch. The Compiler and git push (no variants → no swatches) render their cards cleanly with no empty swatch row.
- **Zero Liquid errors.** `shopify theme check` reports zero new offences on any file this branch edits.

## Stretch Goals

### Stretch A — Active filter count badge

**Implemented** in [blocks/filters.liquid](blocks/filters.liquid). The badge is rendered next to the desktop `Filters` heading in the vertical-style filter rail.

- **Variable reused:** `total_active_values`, computed at [blocks/filters.liquid:31](blocks/filters.liquid:31) and populated by the loop at [blocks/filters.liquid:44-58](blocks/filters.liquid:44) — the top-level computation whose scope covers the whole file, already used by the mobile toggle bubble at [blocks/filters.liquid:341-357](blocks/filters.liquid:341).
- **No counting logic duplicated.** The badge markup was inserted at the desktop `Filters` heading site ([blocks/filters.liquid:115-117](blocks/filters.liquid:115) area) and reuses the existing `.filter-count-bubble` / `.filter-count-bubble__background` / `.filter-count-bubble__text` CSS classes defined at [blocks/filters.liquid:589-628](blocks/filters.liquid:589) — zero new CSS, zero re-computation of the count.
- **Delimited** with `{% comment %}Stretch A ...{% endcomment %}` markers.

### Stretch B — Swatches aren't just colour

**No code change required** — the existing Horizon markup already satisfies the accessibility goal.

- [snippets/variant-swatches.liquid:184-201](snippets/variant-swatches.liquid:184): every swatch `<input type="radio">` carries `aria-label="{{ product_option_value.name }}"`. A screen reader focusing a swatch announces its Grind value name (e.g. "Whole Bean", "Espresso"), not colour only.
- The parent `<label>` at line 178-207 wraps the input, so pointer/keyboard focus paths hit the same aria-labelled input.
- The `.hidden-swatches__count` overflow button at [snippets/variant-swatches.liquid:213-217](snippets/variant-swatches.liquid:213) has its own `aria-label="{{ 'actions.show_all_options' | t }}"` so the "+N" overflow indicator is also announced.

Because the pattern was already correct, this stretch is documented rather than patched — adding another visually-hidden `<span>` alongside the existing `aria-label` would be redundant and would risk double-announcement in some AT combinations.

## Manual Steps

These are my (the developer's) responsibility — Claude Code cannot execute Admin UI, theme-editor, or browser actions. Execute in order.

1. **Search & Discovery.** Shopify Admin → Apps → search "Search & Discovery" → install (or open if already installed). Under **Filters**, target the Roast collection and enable:
   - Product option: `Grind`
   - Product option: `Size`
   - Price
   Save.
2. **Product data prep.** Admin → Products, for each of the four bean products:
   - Debug & Brew, Runtime Error: confirm both `Grind` and `Size` options are populated with all their values.
   - null pointer, Async / Await: confirm `Grind` is populated.
   - Ensure prices differ enough across the four beans that the price_range filter has a real spread (e.g. ZAR 210 / 240 / 275 / 310 — anything monotonic beats identical prices).
3. **Assign swatches.** Admin → Settings → Custom data → Products → Options → `Grind` → edit each option value and assign the hex from Step 1.2:
   - Whole Bean → `#4A2E1A`
   - French Press → `#8B6547`
   - Drip → `#6B4023`
   - AeroPress → `#52321F`
   - Espresso → `#2D1810`
4. **Theme editor config.** Open the Roast collection in the theme customizer (Online Store → Themes → Customize → Collections → Roast):
   - Select the **Filters** block → set `Enable filtering` = on, `Direction` = Vertical, `Show swatch label` = on, `Enable sorting` = on (default). Save.
   - Select the product card → confirm the **Swatches** block is present under the card (add it if not). Save.
   - Select a **Variant picker** block on the product template → confirm `Show swatches` = on. Save.
5. **Local preview & verification.** From the repo root:
   ```
   shopify theme dev
   ```
   Open `http://127.0.0.1:9292/collections/roast` and run every check in the Verification Notes section above — baseline, live-update Network-tab confirmation, swatch end-to-end (grid → product page), pagination / quick-add / no-variant regression checks. Confirm zero new Liquid errors in the preview.
6. **Merge to `main`.** Open a PR in GitHub for `assignment-1.6-collections-filtering-merchandising` → `main`, review the diff, merge via the GitHub UI. Do not push to `main` directly.
