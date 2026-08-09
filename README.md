# El Destro 5000 — website

Deployed from this repo to <https://eldestro5000.com/> via Netlify. Push to `main` and Netlify
republishes in about a minute.

**Do not use Netlify Drop.** A drag-and-drop deploy publishes something Git doesn't know about, and
the next push silently wipes it. Git is the only way in.

## What's in here

```
index.html              the marketing site — a self-contained bundle, everything inlined
chonk-crunch/privacy/   the Chonk Crunch privacy policy page + its fonts
```

### ⚠️ `chonk-crunch/` must not be deleted

**`https://eldestro5000.com/chonk-crunch/privacy/` is published on the Chonk Crunch App Store and
Google Play listings, and both stores re-check that URL after approval.** If it starts returning 404,
that is a store-compliance flag against a shipped app — it can block updates or get the app pulled,
and it fails quietly, so nobody notices until a submission is rejected.

This matters because the site used to be *one file*. Updating `index.html` is safe — it replaces one
file and leaves everything else alone. **Replacing the whole repo contents with a fresh export is
not.** If you ever sync or clear the directory rather than swapping the single file, restore
`chonk-crunch/` before pushing.

Quick check after any deploy that changed the structure:

```bash
curl -sI https://eldestro5000.com/chonk-crunch/privacy/ | head -1   # expect: HTTP/2 200
```

## Updating the marketing site

```bash
git clone https://github.com/el-destro-5000/website.git
cd website
# drop the new index.html export in, replacing ONLY that file
git add index.html
git commit -m "Update site"
git push
```

## Seeing whether a deploy worked

You don't need a Netlify account for this — it surfaces on GitHub:

- **Commit status.** Netlify reports back on each commit and PR; a green
  `netlify/…/deploy-preview` or deploy status means it built.
- **Pull requests get a preview URL** — open a PR instead of pushing straight to `main` and you can
  look at the change on a real Netlify build before it goes live.
- **Rolling back** is `git revert <commit>` + push.

A Netlify account is only needed to read build *logs* when a deploy actually fails.

## The privacy policy page

`chonk-crunch/privacy/` is a static page with no build step, no framework and no JavaScript. Its
three fonts (OFL 1.1, licenses included) are served from this repo rather than Google Fonts, so the
page makes **zero third-party requests** — deliberate, because it is the page that tells users we
share data with nobody but our processor.

**Don't add analytics, a tag manager, a cookie banner or an ad pixel to that route**, even if the
rest of the site gets them site-wide. If tooling injects globally, exclude this path.

Source of truth for the content lives in the game repo under `files/legal/` — including a table
recording where every claim in the policy comes from. Edit it there, then copy the built page here.

## Known issue — the mailing-list signup does not work

The `JOIN THE LIST` form on the homepage is non-functional. Its handler is still the unfilled
template placeholder `{{ onSubmit }}`, and the email input has no `name` attribute — unnamed form
controls are excluded from form data, so the browser submits nothing and the address is discarded.

This lives in the exported bundle, so it needs fixing at the design-tool source or it will survive
the next export.
