# The Vault — a repack-site-style catalog, minus the piracy

A dark, "release archive" styled catalog site. Every card = one item (cover
image, title, description, tags) with a **Download** button that redirects
straight out to wherever you actually host the file (SwiftUpload, Mega,
Gofile, your own server, whatever). This site never stores or serves the
actual files — it's just the front door.

Every file in this project sits in one flat folder — no subfolders at all.

## Files in here

- `index.html` — the site itself
- `style.css` — all the styling
- `app.js` — reads `items.json` and renders the cards
- `items.json` — all your catalog items live in this one file
- `admin.html` — the admin panel (Decap CMS)
- `config.yml` — tells the admin panel what fields to show
- `netlify.toml` — tells Netlify how to deploy this

## How it's built (and why)

You wanted static hosting (Netlify) **and** a real username+password-protected
admin panel. Plain static hosting has no server to check a password against,
so this uses:

- **Admin panel**: [Decap CMS](https://decapcms.org) (the maintained fork of
  the old Netlify CMS) — gives you the "add item, add image, add
  description, add link" form for free, loaded from `admin.html`.
- **Login**: **Netlify Identity** — real email + password accounts, invite
  only, not just a hidden URL.
- **Storage**: every item lives inside `items.json`, right at the repo
  root. When you save in the admin panel, Decap commits the updated
  `items.json` straight to your GitHub repo.
- **Publishing**: that commit triggers a Netlify rebuild automatically.
  There's no separate build step — `index.html` just fetches `items.json`
  directly, so the site updates the moment the deploy finishes (usually well
  under a minute).

**Netlify specifically** (not Vercel or GitHub Pages) is what's needed here —
it's the one with built-in Identity + Git Gateway, which is what makes the
login work without you running a backend.

## One-time setup

### 1. Get these files into a GitHub repo (flat — no subfolders)

- Create a new, empty repo on github.com.
- On the repo page: **Add file → Upload files**, then drag in every file
  from this project — `index.html`, `style.css`, `app.js`, `items.json`,
  `admin.html`, `config.yml`, `netlify.toml`, `README.md` — all at once, so
  they land directly at the top level of the repo (not inside any folder).
- Commit to `main`.
- Double check on the repo's file listing that you see `index.html` etc.
  sitting directly in the list — not a `.zip` and not nested inside another
  folder.

### 2. Create the Netlify site from that repo

- app.netlify.com → **Add new site → Import an existing project → Deploy
  with GitHub** → pick the repo.
- The build settings are already set via `netlify.toml`, so just click
  **Deploy**.

### 3. Turn on Identity

- **Project configuration → Identity → Enable Identity**.
- Under **Registration**, set it to **Invite only**.

### 4. Turn on Git Gateway

- **Project configuration → Identity → Services → Git Gateway → Enable Git
  Gateway**. This is what lets your Identity login actually commit changes
  to the repo.

### 5. Invite yourself

- **Project configuration → Identity → Invite users** → your email.
- Open the email, click the link — it lands on the homepage and redirects
  you into `/admin.html` to set your password.

### 6. Log in and add your first item

- Go to `your-site.netlify.app/admin.html`, log in.
- Open the **Catalog** collection → **Items** file.
- Click **Add "Items"** to add a new card: Title, Cover Image, Description,
  Download Link, Date, Tags.
- **Publish**. Check the **Deploys** tab until it says "Published," then
  refresh the homepage.

## Customizing

- **Site name**: replace `THE VAULT` in `index.html` and `admin.html`.
- **Colors/fonts**: CSS variables at the top of `style.css` (`--bg`,
  `--accent`, `--font-display`, etc.).
- **Remove the placeholder cards**: edit them out of `items.json` directly,
  or delete them from inside `admin.html` once you're logged in.
- **Extra fields** (file size, version number, etc.): add a field under
  `fields:` in `config.yml`, then reference it in `cardTemplate()` in
  `app.js`.

## Local preview (optional)

The admin panel won't work locally (it needs Netlify's Identity/Git Gateway
services), but you can preview the grid layout:

```bash
python3 -m http.server 8080
# open http://localhost:8080
```

## Notes

- Cover images you upload through the admin panel land in this same flat
  folder, referenced by filename from the repo root. If two images end up
  with the same name, the second upload gets a suffix added automatically.
- Nothing here ever handles or stores the actual downloadable files — the
  Download button is a plain link to whatever URL you paste in.
- Only people you explicitly invite via the Identity tab can log into
  `admin.html` — there's no self-signup.
