# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The marketing website for El Destro 5000, a two-person indie game studio. Static HTML, no build
step, no framework, no JavaScript-driven tooling. Deployed from this repo to
<https://eldestro5000.com/> via Netlify — push to `main` and Netlify republishes in about a minute.

## Repo layout

```
index.html              the marketing site — a self-contained bundle, everything inlined
fonts/                   self-hosted webfonts for index.html (OFL 1.1, licenses included)
chonk-crunch/privacy/    the Chonk Crunch app's privacy policy page + its own fonts
```

`index.html` was converted from a Claude Design tool export into hand-written static HTML (see the
comment at the top of the file). The export is archived elsewhere as an input; this file is the only
live copy. A redesign means re-exporting and replacing it, not hand-editing structure blindly.

## Critical constraints

- **Never delete or bulk-replace `chonk-crunch/`.** `https://eldestro5000.com/chonk-crunch/privacy/`
  is published on the Chonk Crunch App Store and Google Play listings, and both stores re-check that
  URL after approval. A 404 there is a store-compliance flag that can block updates or get the app
  pulled — and it fails silently. Only ever swap `index.html` in isolation; never sync or clear the
  whole directory from a fresh export. Verify after any structural deploy:
  `curl -sI https://eldestro5000.com/chonk-crunch/privacy/ | head -1` should read `HTTP/2 200`.
- **`chonk-crunch/privacy/` makes zero third-party requests, deliberately** — it's the page that
  tells users data is shared with nobody but the processor. Its fonts are self-hosted rather than
  pulled from Google Fonts. Don't add analytics, a tag manager, a cookie banner, or an ad pixel to
  this route, even if the rest of the site gets them site-wide; exclude the path if tooling injects
  globally. Content source of truth lives in the game repo under `files/legal/` — edit there, copy
  the built page here.
- **`index.html`'s fonts are self-hosted for the same reason** — the original export pulled from
  fonts.googleapis.com, which leaks visitor IPs to Google before a pixel is drawn. Keep using
  `fonts/`, don't reintroduce a Google Fonts link.
- **Do not use Netlify Drop.** A drag-and-drop deploy publishes something Git doesn't know about,
  and the next push silently wipes it. Git is the only way in.
- **Prefer leaving the repo public.** It holds no secrets (static marketing page + a public privacy
  policy). Flipping visibility can silently sever Netlify's GitHub access — pushes keep succeeding
  and the site keeps serving the last good build, but nothing new deploys, and nothing on GitHub
  turns red to tell you. Fix: recheck Netlify → Site configuration → Build & deploy → Continuous
  deployment, reconnect if needed, then push an empty commit (`git commit --allow-empty`) to trigger
  a rebuild.

## Deploys

Netlify auto-deploys `main` to eldestro5000.com. **Production deploys post no GitHub
status** — a merged PR with no checks means nothing about whether the site updated.
Verify a deploy by fetching the live page and confirming the changed content is there,
never by looking at commit/PR statuses.

Also: changing the repo's visibility silently breaks the Netlify build hook. If deploys
stop landing with no error anywhere, check that first.

The second paragraph is the reason the first one bites — with no status to go red, a broken hook looks exactly like a successful deploy.

## Known issue

The `JOIN THE LIST` mailing-list form on the homepage is non-functional: its handler is still the
unfilled template placeholder `{{ onSubmit }}`, and the email input has no `name` attribute, so the
browser submits nothing. This lives in the exported bundle — fix it at the design-tool source, or the
next re-export will reintroduce it if fixed only here.

## Workflow

```bash
# edit index.html directly, then:
git add index.html
git commit -m "Update site"
git push
```

Prefer opening a PR over pushing straight to `main` — PRs get a Netlify deploy-preview URL so you can
check the change on a real build before it goes live. See Deploys above for how to confirm a push
actually landed. Rolling back is `git revert <commit>` + push.
