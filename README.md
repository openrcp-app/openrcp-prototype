# openRCP Cook Log — clickable prototype

A social cooking log: log what you cooked in 5 seconds from any source, see what your friends are making.
One HTML file, no build step.

## Run it

Open `index.html` in any browser. On a phone it goes full-screen; on desktop it shows a phone frame with a
controls panel beside it.

- Returning-user view: `index.html`
- Brand-new-user view (starts at sign-in): `index.html#onboarding`

## Deploy (Netlify, ~3 minutes)

1. app.netlify.com → **Add new site → Import an existing project** → pick this GitHub repo.
2. Build command: leave empty. Publish directory: `.` (the repo root). `netlify.toml` already says this.
3. Deploy. Every push to `main` redeploys the same URL automatically; branches get their own preview URLs.

Share the Netlify URL (add `#onboarding` for someone new). It works on any phone, no account needed.

## Working on it with Claude

1. Connect GitHub to your Claude account (claude.ai → Settings → Connectors) and make sure it can see this repo.
2. In a new Claude conversation, attach this repo and describe the change in plain English. Screenshots help.
   Claude reads `CLAUDE.md` first, edits `index.html`, commits and pushes. Netlify redeploys in about 30 seconds.
3. Want to see a change before it goes live? Ask Claude to put it on a branch and open a pull request;
   Netlify gives the branch its own preview link.

Example prompts that work well:
- "Make the invite link row smaller and move it under the contact list."
- "Add a recipe from Smitten Kitchen to the catalog with its real cover photo."
- "On the recipe page, show the source name before the author."
- "Undo the last change to the feed."

`CLAUDE.md` holds the product decisions and a map of the file. Keep it current when a decision changes.

## Structure

```
index.html     the whole app: styles, markup, data, views, sheets, events
photos/        24 recipe covers, 800×600 JPEG, named by recipe id (r1.jpg … r24.jpg)
CLAUDE.md      product rules + file map, read by Claude on every session
netlify.toml   publish from repo root, no build
```
