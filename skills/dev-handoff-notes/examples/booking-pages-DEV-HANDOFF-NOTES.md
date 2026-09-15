# Openslot booking pages — dev handoff notes

> **How to read this.** The prototype shows the behaviour: **https://openslot-booking.example.com**. The repo has the styles and code. This doc is the annotation layer: the rules, states and edge cases that aren't obvious from clicking through, what is mocked, and what is not built. Rationale is not in here.
>
> **Scope.** Booking pages — the list, a page's settings, the availability editor, the public booking flow an invitee sees, and the booked/cancelled states. Team round-robin pages, paid bookings and the admin area are separate work.
>
> **Last updated:** 2026-09-15.

## At a glance

| Area | What exists |
|---|---|
| The model | A booking page = one meeting type + its availability + its questions. Availability is a weekly pattern plus date overrides. |
| Availability rules | Overrides beat the weekly pattern. Buffers, minimum notice and daily cap narrow what an invitee can pick. |
| Timezones | Stored in UTC. Owner edits in the page's zone; the invitee sees their own, detected and switchable. |
| Pages list | Search, `Bookings this month`, live/paused toggle per row, copy link. |
| Page settings | Details → Availability → Questions. Every change saves on blur. |
| Availability editor | Weekly grid with per-day time ranges; a calendar for date overrides. |
| Public booking flow | Pick a date → pick a time → answer questions → booked. |
| Booked / cancelled | Confirmation, reschedule and cancel, each with its own page and email. |

---

## The model

- A **booking page** = a meeting type (name, length, location) + its availability + the questions an invitee answers. One page produces many bookings.
- **Availability is a weekly pattern plus date overrides.** The pattern repeats indefinitely; an override replaces a single calendar date outright, including making it unavailable.
- **An override always wins over the pattern**, whether it adds or removes time.
- **Three rules narrow what an invitee sees**, applied in this order: **minimum notice** (how soon the earliest time can be), **buffers** (protected time before and after each booking), **daily cap** (how many bookings that date will accept).
- **A time the owner is already busy at is never offered.** Availability says when the owner *can* meet; the calendar says whether they are free.
- **Pausing a page keeps its link alive** and shows the paused state to anyone who opens it. Existing bookings are unaffected.
- **Deleting a page does not cancel its bookings.** Those keep their own reschedule and cancel links; the page's link stops working.

## Availability rules

- **Weekly pattern:** per weekday, any number of time ranges, or off. A day with no ranges is unavailable.
- **Overlapping ranges on one day merge.** `9:00–11:00` plus `10:00–12:00` becomes `9:00–12:00` and the editor shows the merged range.
- **A range must end after it starts.** An end time at or before the start is rejected in place: `End time must be after the start time.`
- **Buffers are per page:** `Before event` and `After event`, each 0–120 minutes. A time that availability allows is still withheld when a buffer overlaps it.
- **Minimum notice** is 0 minutes to 30 days. Times inside it aren't offered, including on today. ⟨confirm: whether minimum notice counts clock hours or the owner's available hours⟩
- **Daily cap** is 1–20 or off. A date at its cap offers no times and reads as fully booked.
- **Offered times step by the meeting length**, from the start of each range. A 45-minute meeting in `9:00–11:00` offers 9:00 and 9:45 — never a time that runs past the range.
- **Overrides are per date**, set in the editor's calendar: either replacement ranges, or unavailable. An override on a past date is kept but never consulted.

## Timezones

- **Every time is stored in UTC** and rendered in the reader's timezone.
- **The owner edits availability in the page's timezone**, stated once above the weekly grid. Changing the page's timezone keeps the ranges as entered and shifts what they mean.
- **The invitee's timezone is detected from the browser**, named in a control at the top of the booking flow, and switchable. Every time shown carries it.
- **Confirmation, reschedule and cancel pages show the meeting in the invitee's timezone**, with the owner's beside it when the two differ.

## Pages list

- Rows: name · meeting length · location · link · `Bookings this month` · a live/paused switch · `⋯`.
- **Search** matches the start of any word in the page's name.
- **Copy link** copies the public URL and the button reads `Copied` for 2 seconds, then reverts.
- The live/paused switch is on the row — no confirm. Pausing shows a toast with **Undo**: `"<name>" is paused.`
- Row `⋯`: Edit · Duplicate · Copy link · Delete.
- Duplicate produces `<name> (copy)`, paused, with a fresh link and no bookings.
- **Delete confirm:** *"Anyone with this link won't be able to book. Bookings already made are kept, and invitees can still reschedule or cancel them. This can't be undone."*
- **Empty state:** *"No booking pages yet. Create one and share its link."* + `New booking page`.

## Page settings

- Three sections in order: **Details → Availability → Questions.**
- **Details:** `Name` · `Length` (15 / 30 / 45 / 60 minutes, or `Custom` revealing a minutes field) · `Location` (`Video call` / `Phone` / `In person` / `Custom`, each revealing its own field) · `Description`.
- **Every field saves on blur**, with a `Saved` marker beside the section heading for 2 seconds. There is no Save button.
- **A name is required.** Clearing it and leaving the field restores the previous name and flags the field: `Give this page a name.`
- **Questions:** `Name` and `Email` always exist, always required, and can't be removed or reordered. Added questions are short text, long text, single select or checkbox; each is required or optional and can be reordered by dragging.
- **A select question needs at least one option.** Saving with none flags it: `Add at least one option.`
- **Removing a question keeps it on bookings already made.** The confirm says so: *"Answers already collected stay on those bookings."*

## Availability editor

- **Weekly grid:** one row per weekday, each with its ranges, `+ Add time` per day, and a copy control — `Copy times to…` with weekday checkboxes.
- A day toggled off keeps its ranges greyed and restores them when toggled back on.
- **Overrides calendar:** one month at a time. A date with an override is marked; clicking one opens its ranges, or `Mark unavailable`. `Clear override` returns the date to the weekly pattern.
- **Times are entered in 15-minute steps.**
- **A page with no available time anywhere** shows a caution notice: *"This page has no available times, so nobody can book it."* It saves and stays live.
- **Nothing is blocked while editing.** Conflicting entries flag in place and the rest of the page keeps saving.

## Public booking flow

Three steps on one page.

- **Step 1 — pick a date.** A month calendar with bookable dates enabled and everything else disabled; the first bookable date is preselected. `<` and `>` move by month; `>` is unavailable past the booking window.
- **Step 2 — pick a time.** The chosen date's times as a list. **No times that day:** *"No times available on this date."* **No times in the whole month:** *"Nothing available this month."* + `Next month`.
- **Step 3 — answer the questions.** Name and email first, then the page's own questions in their order. Required fields flag on submit, not on blur.
- **The chosen time is held for 10 minutes** while the invitee fills the form. The remaining time is not shown. If it expires and someone else has booked the time, submitting returns to step 2 with: *"That time was just booked. Pick another."* If the time is still free, submitting books it.
- **A time taken by someone else disappears on the next step-2 view** rather than erroring.
- **Switching timezone** re-renders every time shown and keeps the chosen time.
- **A paused page** shows *"This page isn't taking bookings right now."* and nothing else. **A deleted page's link** shows the same generic not-found page as any bad link.

## Booked, rescheduled, cancelled

- **Confirmation:** the meeting summary, `Add to calendar` (`.ics` download), and the reschedule and cancel links, with a note that they're also in the email.
- **Reschedule** re-enters the flow at step 1 with the questions already answered and not asked again. The old time frees only once the new one is confirmed.
- **Cancel** asks for an optional reason, then confirms in place.
- **A booking in the past can't be rescheduled or cancelled**; both links show *"This meeting has already happened."*
- **An already-cancelled link** shows the cancelled state rather than an error.

**Mocked:** the confirmation email and the `.ics` file's calendar invite.

## Responsive

| Breakpoint | Change |
|---|---|
| <1024px | list drops `Bookings this month` |
| ≤768px | the availability editor's weekly grid stacks one day per row; the booking flow's calendar and time list become separate steps rather than side by side |

- **The list never drops the name, the switch or the `⋯`.**
- The booking flow is laid out for phone width first.

## Motion rules

Values live in the prototype's code; these are the rules:

- Animate `transform` and `opacity` only. Entrances start at scale 0.97 or higher, with opacity.
- Popovers grow from their trigger edge.
- The step-to-step move in the booking flow is a short cross-fade, not a slide.
- Exits are faster than entrances and share the entrance direction.
- `prefers-reduced-motion` removes travel and keeps opacity.

## Accessibility

- **The date calendar is a grid**, arrow-key navigable, with unavailable dates disabled and announced as such.
- **Each step announces itself** when it becomes current. The flow is one page; the step change is spoken.
- **Validation messages are tied to their field** and announced on submit.
- **No disabled primary control.** `Confirm` validates on click and moves focus to the first problem field.
- **The timezone control states the current zone in its accessible name**, not just the abbreviation.
- Times are radio buttons, not links.
- **The focus ring means keyboard focus only.** Selected state is the control's own fill.
- `⋯` buttons have item-scoped names (`More actions for <name>`).

## Not built in this prototype

Out of scope here; absent, not simulated.

- **Calendar connection.** Busy times come from a fixture; nothing reads a real calendar.
- **Team and round-robin pages.** One owner per page.
- **Paid bookings.** No payment step anywhere.
- **Recurring meetings.** One booking is one meeting.
- **Booking-window settings** (how far ahead an invitee may book). The flow enforces a fixed 60 days.
- **Notifications and reminders** beyond the confirmation email.

## What the prototype fakes

- **All data lives in the browser** (`localStorage`): pages, availability, overrides and bookings. No API. **Reset prototype** returns to the seed.
- **Busy times** are a fixed fixture; the same times are always taken.
- **Bookings this month** is a seeded number and doesn't change when a booking is made in the prototype.
- **The invitee** is whoever fills the form; there are no accounts on the public side.
- **Emails** send nothing. The `.ics` file downloads but carries placeholder organiser details.
- **The 10-minute hold** is enforced in the browser only; two tabs can double-book.
