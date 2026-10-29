# openRCP Cook Log — prototype

Read this before changing anything. It holds the product decisions already made, so edits from different people
(and different Claude sessions) build the same product.

## What this is

A clickable prototype of a social cooking log: a 5-second "I made it" from any recipe source, plus a feed of what
your friends actually cooked. Think Letterboxd/Goodreads for home cooking, built on openRCP (openrcp.com).

It is one self-contained file, `index.html`, plus `photos/`. No build step, no framework, no dependencies beyond
Google Fonts. Open the file in a browser and it runs. Nothing is saved between reloads; all data is seeded in the file.

## Product rules (do not undo these without a conversation)

**Catalog model (the "ISBN")**
- A catalog entry is title, author, source, cover photo, canonical URL, time. Never recipe text. We never host or
  display a recipe's ingredients or steps as a public page.
- Subscription sources (NYT Cooking) open at the source. Blogs open in openRCP's ad-free reader (on-demand, for the
  signed-in user only). Cookbook entries show a page number and have no link-out; their cover comes from the
  publisher's online version of the same recipe.
- Every catalog tile uses the source's own photo. No placeholders, gradients, emoji or illustrations, ever. The only
  photo-less state is a recipe the user typed in by hand (shown as a plain tile with a camera glyph).

**Logging**
- The share sheet is the main door (share extension on iOS, share target on Android): from NYT Cooking, Safari,
  Instagram, anywhere. Paste a link is the same path by hand. Search is for cookbooks and things you know by name.
- The camera is never a primary CTA. Photo, rating, note and date are all optional on the Made it sheet.
- "Today" on the Made it sheet is a date picker (last 7 days as chips, native date field for older).

**Feed**
- One feed, no modes. Friends' events first (made / saved / is making this Monday), then an unbounded "Trending now"
  stream with the same infinite-scroll mechanics. The feed never dead-ends.
- "Nothing from friends yet" is a single muted line with a Find friends link and an ✕, not a card.
- No "cook along". A friend's plan is information, not an invitation.
- Trending signals (NYT trending, TikTok, Instagram, friends of friends, openRCP) are simulated in the prototype.
- The taste test ("Tune your feed", swipe 8 cards) lives in the feed as a dismissable card in the third slot. It is
  not an onboarding step.

**Onboarding** (welcome → sign in → handle → what you eat → friends → first log)
- Sign in: Continue with Apple, Continue with Google, Continue with email (magic link, no password). Buttons follow
  each platform's guidelines; the brand marks are rendered by the real SDK buttons in production, not drawn here.
- Progress dots start after sign-in. Every step has Back. The primary CTA is pinned in the viewport.
- "What you eat" is dietary restrictions only: tinted line-icon chips, one tap each, friends never see it.
- Friends: Sync contacts row, compact invite-link row, Follow for people on openRCP, Invite for people who aren't.
- First log: pick a source (radio chips), search, multi-select, "Log 3 cooks".

**My Kitchen** (the user's profile)
- Stats row; following and followers are tappable. "Planning to cook": the next 7 days, up to 4 recipes per day,
  multi-select picker. Not a meal planner: no slots, no groceries, no calendar. Logging a planned recipe clears it.
- Cooks and Want to cook use the same 3-up photo grid. A friend's profile is the same screen, read-only, with their plan.

**More**
- Only real things: Friends, Notifications (per-event on/off), Settings, Help and feedback (mailto support@openrcp.com).
- No Paprika/Mela import, no export, no "connected sources".

**Recipe page**
- Compact "Open in NYT Cooking" row (same component as feed cards, `openlink primary`). I made it / Want to cook /
  Plan a day. One "Friends" section (making it, then made it). "Similar recipes" carousel. No "Your history" section.

**Voice**
- Direct, short, no exclamation marks. Welcome: "So, what are your friends cooking?" / "All the food photos. None of
  the kid photos." Invite text: "I'm on openRCP. Come see what recipes I'm cooking 🍳".

## Design language (openRCP)

- Tokens live in `:root` at the top of `<style>`: background `#FFFEFA`, ink `#1F1A14`, cream `#F4F0E7`,
  primary green `#01281A`, radius 8–12px. Dark mode is supported via `prefers-color-scheme` and `[data-theme]`.
- Type: Kaisei Decol (display, headings) and Instrument Sans (body), loaded from Google Fonts.
- Logo: the openRCP "R" monogram, inline SVG in `ICON.logo`.
- Components to reuse rather than invent: `.btn` (primary / secondary / ghost / sm), `.chip` and `.rchip` (radio),
  `.seg`, `.search`, `.field`, `.rrow` (list row), `.tile` (photo tile: sm / md / tr / wide), `.card` and
  `.card.lite`, `.method` (icon row), `.openlink`, `.sheet` with `sheetHdr()`, `.callout`, `.nudge`, `.toast`.

## Where things are in index.html

Everything is in one file, in this order:

1. `<style>` — tokens, then components roughly in the order the screens use them. Search for a class name.
2. Markup — the phone frame (`#phone` → `#views`, `#tabbar`, `#overlay`) and the prototype controls `<aside>`.
3. `<script>`, in sections marked with `/* ---------- Name ---------- */`:
   - **Seed data**: `SOURCES`, `RECIPES` (24 entries, ids r1–r24; `k` = concept keywords from `KEYWORDS`), `SIGNALS`,
     `CONTACTS`, `SYNTH_NAMES`, `USERS`, `seedCooks()`, `seedFriendWants()`, `seedFriendPlans()`, `freshState()`.
     All app state lives in the single object `S`.
   - **Small components**: `ICON`, `cover()`, `tile()`, `avatar()`, `stars()`, `hdr()/brandHdr()/backHdr()`,
     `dayLabel()`, `weekDays()`, `plannedFor()`, `dayTile()`.
   - **Views**: one function per screen on the `views` object — `feed` (+ `card`, `socialCard`, `catalogCard`,
     `feedBatch`), `explore` (search + trending, `trendBatch`), `diary`/`user` (both call `profile()`), `follows`,
     `recipe`, `cook`, `friends`, `me` (More), `notifprefs`, `settings`, `notifs`, `reader`, `onboard`.
   - **Navigation**: `render()`, `go()`, `back()`, `setTab()`; browser history is mirrored so the phone's back gesture
     works. Infinite scroll is wired in `render()`.
   - **Sheets**: `openLog`, `openMade`, `openShare`, `openOAuth`, `openEmail`, `openInvite`, `openPaste`,
     `shareDemo`/`demoSheet`, `openTaste`, `openPlan`/`openPlanPick`/`openPlanDay`/`openFriendDay`, `setSynth`.
   - **Events**: ONE delegated click handler on `#phone`. Every tappable thing is a `data-*` attribute
     (`data-recipe`, `data-madeit`, `data-go`, …). To add an action: add the attribute to the markup, add it to the
     selector list at the top of the handler, add an `if(ds.yourAttr)` branch.
   - **Boot**: `#onboarding` in the URL starts from the welcome screen; otherwise it boots as a returning user.

## How to make changes

- Keep it one file. Keep it dependency-free. Keep the phone frame at 390px wide and everything inside it in the viewport.
- Add photos to `photos/` as 800×600 JPEG, named by recipe id (`r25.jpg`), always the source's own cover.
- After editing, open `index.html` and click through the flow you touched. The "Flows to try" list in the controls
  panel is the regression checklist. Check dark mode too (OS setting or `document.documentElement.dataset.theme='dark'`).
- Prototype controls (the panel beside the phone) are for demoing: share handoff, digest push, restart onboarding,
  reset, synthetic cooks. They are not product.

## What the prototype is for

Showing the product to people and deciding what to build. Covers are the publishers' photos and are fine for a
private prototype shared with a few people; this is not a public launch asset.
