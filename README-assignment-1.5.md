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

Rebuilt the same two settings at the **section scope** on `sections/cart-drawer-section.liquid` (schema `settings` array), then swapped the snippet reads from `settings.…` to `section.settings.…`.

**Which is actually correct: global.** My first instinct was global and it stayed correct after the exercise. Reasoning:

- The threshold is a business rule (shipping cost break-point), not a UI dial. It should be set once, in one place, and read wherever the store surfaces the number.
- Section-scoping puts the setting on the drawer only. The `/cart` page would need its own copy (added to `sections/main-cart.liquid`) and the two would drift as soon as one merchant updates one and forgets the other. Two sources of truth for one policy.
- Section-scoped settings live in template JSON (`templates/*.json`), which Shopify's GitHub sync writes back on every merchant edit. Global settings live in `config/settings_data.json`. Both sync, but the global one is a single file the merchant knows to look at; the section-scoped one is buried inside whichever template happens to include the section.
- Global also plays nicely with the enable checkbox — flipping one switch turns the feature off everywhere.

The section-scoped version is preserved on this branch so both live implementations can be compared side by side; the shipping snippet reads from the section-scoped setting when present and falls back to the global.

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
