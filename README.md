# Meet — Centre for British Progress

A static meeting booking page for the Centre for British Progress core team. Hosted on GitHub Pages at [meet.britishprogress.org](https://meet.britishprogress.org).

## How it works

1. **Select people** — visitors pick 1-3 team members from a grid of 14 core staff
2. **Choose a method** — In-person (80 Strand office or another location) or Call (video call / phone call). For 2+ people, only video call is available (no phone option)
3. **Book or email** — if a Google Calendar booking page exists for that combination, it loads in an embedded iframe. Otherwise, a pre-filled email draft is shown with a "Send Email" button that opens Gmail compose

## URL scheme

The page uses hash-based URLs to persist state:

```
index.html#julia                          → selects Julia, goes to stage 2
index.html#david-julia-and-maxx           → selects David, Julia & Maxx
index.html#julia?method=office            → selects Julia + 80 Strand
index.html#david-and-julia?method=video-call → selects David & Julia + video call
```

**Name slugs:** first names, lowercased. If two people share a first name (e.g. Matthew Stubbs & Matthew Grant), a last initial is appended: `matthew-s`, `matthew-g`.

**Method values:** `office`, `in-person`, `video-call`, `phone-call`

## Booking URL config

`booking-urls.json` maps URL slugs to actual Google Calendar booking page URLs. The slug format is `{people}-{method}` matching the subdomain pattern `meet-{slug}.britishprogress.org`.

- If a slug has a non-empty value → the iframe loads that URL
- If the value is `""` → the email draft fallback is shown

To add a new booking page, set the slug's value to the full Google Calendar URL.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire site — HTML, inline CSS & JS |
| `booking-urls.json` | Maps URL slugs to booking page URLs (or `""` for email fallback) |
| `CNAME` | Custom domain config for GitHub Pages |
| `404.html` | Custom 404 page |
| `fonts/` | National 2 font files (matching britishprogress.org) |
| `logo.png` / `logo-full.png` | Centre for British Progress logo |
| `favicon*` / `apple-touch-icon.png` | Favicons (light & dark variants) |
| `site.webmanifest` | PWA manifest |

## Tech

Plain HTML + CSS + vanilla JS. No build step, no dependencies. Single `index.html` file with everything inlined.
