# Dev Coffee — Build Your Own Storefront

Shopify Theme Development Module · Day 1 · Store Setup & Development Environment

**Change notes.**

1. **Theme migration.** Originally attempted with Shopify Dawn on a store named `my-store-yofcpcr4`. Per instructor guidance (Skye, Bitcube SDTP), migrated to Horizon.
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
