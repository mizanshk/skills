# Lantern Promo Codes — dev handoff notes

> **How to read this.** The prototype shows the behaviour: **https://lantern-promo-codes.example.com**. The repo has the styles and code. This doc is the annotation layer: the rules, states and edge cases that aren't obvious from clicking through, what is mocked, and what is not built. Rationale is not in here.
>
> **Scope.** Promo codes only — the list, detail page, discount editor, targeting, the checkout preview and the post-publish moment. `/orders`, `/customers` and `/mocks/*` in the repo are another project's skeleton. `/catalog` is the product-catalog prototype vendored in for navigation and has its own handoff.
>
> **Last updated:** 2026-09-15.

## At a glance

| Area | What exists |
|---|---|
| The model | One Code object with a list of targets. Target kinds: product, collection. Precedence: product beats collection. |
| Targeting & schedule | Schedule per assignment. Collection: whole-day dates only. Product: dates with times. Conflict = two codes on one product with overlapping schedules. Suggested codes. |
| Codes list | Word-prefix search, `Modified` (edits to the code only), Draft badge, empty state → templates. |
| Code detail | Analytics (windowed) → Discount → Applied to. Primary: **Apply** (Product / Collection). Conflicts: one banner + per-row marks. |
| Composing a discount | A rule stack: one discount type, optional minimum, optional limits. Stacking on or off. |
| The editor | Focus mode. Live checkout preview; panel with code level and rule level; Publish validates on click; undo. |
| Templates | 9 in 3 groups; typical redemption rate where measured; reuse-first when adding to a product. |
| Checkout preview | The banner, the code field and the line-item discount as a customer sees them. |
| Post-publish moment | Status line + one-tap chips: curated codes, else fitted templates, else Browse templates. |
| Results loop | Copy link → simulated first redemption → toast + bell. |

---

## The model

- A code = **type + discount + limits + targeting**. One object. No "storewide code" or "product code" variants.
- **Targets are products and collections.** No tag targets, no customer-segment targets, no storewide default.
- **Precedence: product beats collection.** Two tiers.
- **A collection target matches the collection and its sub-collections.**
- **Picking a vendor in the collection picker writes one collection target per vendor collection**, each marked as via that vendor. Picking the same vendor again adds nothing.
- **Deleting a code removes it from every product it's applied to.** Carts holding the code drop the discount at their next recalculation.
- **A code's name** is set by Rename on the detail page. A new code with no name takes its label, else `Untitled code`.
- **Codes are visible to everyone with catalog access.** The owner edits; anyone with access applies. ⟨confirm: whether "apply" rides the existing "edit products" permission⟩

## Targeting & schedule

- **Schedule is per assignment:** a `Starts` date and an optional `Ends` date. Collection assignments take whole days only. Product assignments take a date and a time.
- **A code with no `Ends` runs until removed.** The row reads `never expires`.
- **Conflict = two codes on the same product with overlapping schedules.** Collection-vs-product is never a conflict.
- **A code applied directly to a product overrides one it inherits from a collection.** There is no other override mechanism.
- **A code can be *suggested* to a vendor instead of assigned.** A suggested code never counts for coverage, precedence or conflicts. It surfaces as a one-tap chip on that vendor's products; tapping writes a direct product target.
- Two label sets exist for the same thing: `Starts / Ends` (detail page) and `From / Until` (checkout preview). ⟨confirm: one vocabulary⟩

## Codes list

- Columns: code · name · `Applied to` · `Redemptions` · `Redemption rate` · `Modified`. Sorted by Modified, newest first.
- **Search** (placeholder `Search`) matches the **start of any word** in the code or name. No results: `No codes match` + `Clear search`.
- **`Modified` = the last edit to the code itself.** Applying or removing targets, changing a schedule and redemptions do not update it. Exact timestamp on hover.
- **Draft** badge on codes that have never been published.
- **Create** and the empty state's `Create a code` open the template picker.
- Row `⋯`: Edit · Duplicate · Apply · Delete.
- Duplicate → `Copy of <name>`, no targets, zero redemptions, toast `Code duplicated.`
- Delete confirm (`Delete code`): *"This code will be removed from every product it's applied to. Carts using it lose the discount. This can't be undone."* → toast `Code deleted.`

## Code detail

- Order: **Analytics → Discount → Applied to.**
- Header: name · `Last modified: <relative>` (exact time on hover) · **Apply** (menu: *Product* / *Collection*) · `⋯`: Rename (`Rename code`) · Duplicate · Delete.
- **Analytics windows:** `Last 7 days` · `Last 30 days` · `Last 90 days` · `All time`. Default `Last 30 days`. Tiles `Seen` → `Redeemed` → `Redemption rate`, all reading the selected window; the window is stated once, in the control.
- **Redemption rate = redemptions ÷ carts that saw the code**, one decimal.
- **Analytics empty state** — only when lifetime seen is zero: *"Once this code starts appearing at checkout, its redemptions appear here."* A zero inside a window is a value, not an empty state.
- **Discount card:** preview · `Discount` row · `Limits` row · **Edit** (ghost) → the editor. Discount reads `<20% off | $15 off | Free shipping>`. Limits is one line: `<Stacks | Doesn't stack>, <one per customer | unlimited per customer>` — both halves always present.
- **Applied to — empty:** *"This code isn't live anywhere yet."* + `Choose where it applies` → the collection picker.
- **Applied to — rows:** kind icon · name (linked) · schedule · `Remove`. A conflicting row carries ⚠ with the tooltip `Overlaps with "X" from <date>`. A suggested row carries a `Suggested` badge and no schedule control.
- **Conflicts banner:** caution style, one line, title only, above the rows: `One product has two codes at the same time, so only one will apply` / `<N> products have two codes at the same time, so only one will apply`.
- **Delete confirm states the blast radius**, computed live, then `This can't be undone.`: *"No cart is using this code right now."* / *"It's live on 7 products right now."* + *"Carts keep their other discounts."* / *"4 of them fall back to a collection code; the rest have no discount."*

### Target picker (Apply → Product / Collection)

- Title `Apply to a collection` / `Apply to a product`, no subtitle. Search (`Search collections…` / `Search products…`) · `Filters` pill · one flat list · footer `Nothing selected` / `<N> products selected` · **Done**.
- **Filters:** collections by *Type* (`All` / `Seasonal` / `Evergreen`) and *Vendor*; products by *Vendor* and *Collection*. Values open in a panel beside the properties panel, which stays open with the active row marked. Vendor and Collection values have a search field; Type doesn't. The pill label never changes; applied filters render as removable chips.
- **Select-all** (`Select all products`) = everything matching the current search and filter together, minus conflicting rows. Mixed → click clears. When every visible row conflicts: toast `Every product here already has a code for those dates.`
- **A conflicting row** stays in the list, unavailable but focusable, with the caption `Already has "X" for these dates`; clicking it toasts `<Product> already has "X" from <date>.`
- Picking toggles the target immediately; Done closes.

## Composing a discount

- **Discount type:** `Percentage off` · `Fixed amount off` · `Free shipping`. One per code.
- **Minimum:** `No minimum` (default) · `Minimum spend` (reveals an amount field) · `Minimum items` (reveals a count field).
- **Limits:** `Uses per customer` (`1` default · `Unlimited`) · `Total uses` (blank = unlimited).
- **Stacking:** `Stacks with other codes` toggle — tooltip *"Customers can combine this with another code at checkout. Off makes it exclusive."* Default off.
- **Free shipping ignores Minimum items.** The field is hidden when Free shipping is selected.
- **Percentage off caps at 100.** Typing more sets 100.
- **Fixed amount off never makes a line item negative.** The preview shows the item at `$0.00`.
- **Checkout banner copy** is one editable line, default `Use code <CODE> for <discount>`. The `<CODE>` and `<discount>` tokens render as pills and always resolve.

**Mocked:** currency is fixed to USD; tax is not modelled.

## The editor

- **Focus mode** at `/codes/focus/:id` (and `/new`), no sidebar. Header: `Edit code` / `New code` · ✕ · **Undo** · **Publish** (`Publish code` when creating). ✕ and Publish both return to the page you came from. No name field.
- In create mode with targets already chosen, a line under the preview reads `Will apply to <target>`.
- **Undo** is always enabled — tooltip `Undo (⌘Z)` / `Nothing to undo`. ⌘Z / Ctrl+Z does the same.
- **Layout:** a panel and a live checkout preview. ≤768px the preview stacks above the panel.
- **Code level** (panel): `Code` (the string, `Generate` button) · `Discount` · `Limits`. **Rule level** (select a rule in the preview; `Back to code` exits): the rule's own fields, with `Remove rule`.
- **`Generate`** produces an 8-character uppercase code. Typing a code lowercases nothing; codes are stored as typed and matched case-insensitively.
- **Publish is never disabled.** On click: empty code string → *"Add a code before publishing."*; a code already in use → *"That code is taken. Try another."*; a percentage of 0 → *"Set a discount above 0."* None is blocked from being fixed in place.
- **Draft** state: a code saved without publishing. Drafts never appear at checkout.
- **Publish toasts** `Code published.` / `Changes published.`

**Mocked:** the "taken" check runs against the seed only.

## Templates

- **9 templates, 3 groups:** *Acquisition* — First order · Newsletter welcome · Referral thanks. *Retention* — Come back · Birthday · Loyalty tier. *Clearance* — End of season · Bundle and save · Last units.
- Picker title `New code`; each template shows `Typical redemption rate` **only where a measured figure exists** (four templates). `Use template` · `Start from scratch`.
- **Adding a code to a product** opens a picker of existing codes with `Create new code` → the template picker. **With zero codes in the account, the template picker opens directly.**

## Checkout preview

- Shows the storefront banner, the code field and one sample cart with the discount applied.
- **Banner:** the editable line from the discount card. Hidden when the code is a draft.
- **Code field states:** empty (`Promo code`) · applied (`<CODE> applied` + `Remove`) · invalid (`That code isn't valid`) · expired (`That code has expired`) · not-stackable (`Remove <OTHER> to use this code`).
- **The sample cart** is fixed: two line items and shipping. Free shipping strikes the shipping line.

**Mocked:** the cart; there is no storefront.

## The post-publish moment

- **`/codes/:id/published`** shows once, immediately after the first publish. Not in navigation.
- Content: `Your code is live` · the code · the checkout preview · a status row · **Copy link** (primary) · **Open editor**.
- **Status row, targets present:** *Applies to **<N> products**, via <collection>* (the clause only when inherited).
- **Status row, no targets:** *"Add where it applies. Right now nobody can use it."* + chips.
- **Chips, three tiers:** curated codes the vendor suggested, caption `Suggested by your vendor` — tap applies as a direct product target and toasts `"<name>" now applies to this product.`; else two **fitted templates** with a reason line (`This looks like a <category> product. <reason>`) — tap opens the editor with the template loaded; else a single `Browse templates` chip.
- **Chips show only when the code has no targets.**

## Results loop

- Copying the code's share link → **8 seconds** later a simulated customer redeems: toast *"Jordan Vale used FIRST10 on a $64 order"*; the bell shows an unread count; the panel (`Notifications`) lists the event and links to the code. Opening the panel marks all read.
- **The redemption increments Seen and Redeemed** on the code.

**Mocked:** the customer, the order, the delay.

## Responsive

| Breakpoint | Change |
|---|---|
| <1280px | list drops `Modified` |
| <1024px | sidebar collapses to a rail; list drops `Redemptions` and `Redemption rate` |
| ≤768px | hamburger drawer; list drops `Applied to`; editor stacks — preview above, panel below |

- **The list never drops the code or the `⋯`.** Headers hide with their cells.

## Motion rules

Values live in the prototype's code; these are the rules:

- Animate `transform` and `opacity` only. Entrances start at scale 0.97 or higher, with opacity.
- Popovers and tooltips grow from their trigger edge. Menus open below; flip above only when below doesn't fit; when neither fits, take the roomier side and scroll inside.
- **Tooltips dwell ~400ms before showing. Within ~300ms of one closing, the next shows instantly with no entrance.**
- Toasts slide in from the edge and auto-dismiss after 10s; exits are faster than entrances.
- Metric tiles roll like an odometer on window change.
- Nothing keyboard-repeated animates. `prefers-reduced-motion` removes travel and keeps opacity.

## Accessibility

- **A menu is one tab stop.** Arrow keys move between items, Tab leaves, Escape closes and returns focus to the trigger. Escape closes modals.
- **Tooltips show on keyboard focus** and are announced through the trigger, except tooltips that only repeat the trigger's own name (icon buttons), which are visual-only.
- **No disabled primary control.** Publish, Apply, Undo stay enabled and explain themselves.
- **Unavailable rows stay in the tab order** and explain themselves (conflicting rows in the pickers). Never `disabled`.
- Target-picker rows are multi-select checkboxes to assistive tech.
- The conflicts banner is a status, not an alert.
- **The focus ring means keyboard focus only.** Open/pressed state is the control's own fill.
- `⋯` buttons have item-scoped names (`More actions for <name>`), not tooltips. Metric tiles expose their real value to screen readers.

## Not built in this prototype

Out of scope here; absent, not simulated.

- **Customer-segment targeting.** Codes target products and collections only.
- **Import / export of codes.**
- **Scheduling by time zone.** All times are the store's time zone.
- **Multi-currency.** USD only.
- **Unsaved-changes guard in the editor.** Leaving with edits pending is not intercepted.
- **The storefront.** Only the checkout preview exists.

## What the prototype fakes

- **All data lives in the browser** (`localStorage`): codes, targets, drafts; notifications in a second key. No API. **Reset prototype** returns to the seed.
- **The catalog** is a static set of ~300 fabricated products with fabricated vendors and stock images.
- **Analytics** are seeded lifetime totals with a generated daily series behind the window control.
- **Typical redemption rate** on templates and the fitted-template reasons are fixed figures, not live data.
- **Results loop**: simulated customer, order and delay; the analytics increment is real.
- **Copy link** copies nothing real.
- **The floating `Prototype controls` panel** (Empty / Filled · Reset prototype) is a demo control, not design.
- **`/mocks/*`, `/orders`, `/customers`**: another project's skeleton. Ignore.
