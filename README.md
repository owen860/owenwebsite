# Waitlist page

A one-screen waitlist page: dark hero, glowing email field, product photo on a lit
floor. Signups go straight to Klaviyo. No build step, no dependencies — one HTML
file and three assets.

```
netlify.toml          publish = "public"; this folder is the repo root, so no base directory
public/
  index.html          the whole page: markup, CSS and JS in one file
  fonts.css           BROmega 400/600/700, embedded as base64 (see "Fonts" below)
  monitors.webp       hero photo, 1379px wide, transparent background
  monitors-800.webp   the same photo at 800px, served to phones
```

## Deploy

Push this to a GitHub repo, then in Netlify: **Add new project → Import an existing
project**, pick the repo, and accept the defaults:

| setting | value |
| --- | --- |
| Branch to deploy | `main` |
| Base directory | *(leave empty)* |
| Build command | *(leave empty)* |
| Publish directory | `public` |

`netlify.toml` already sets the publish directory, so in practice you can just click
through. Every push to `main` redeploys. HTTPS is issued automatically; add a custom
domain under **Domain management**.

Vercel, Cloudflare Pages and GitHub Pages work equally well — it is plain static
files. Only the publish directory (`public`) matters.

## Connect Klaviyo

Open `public/index.html`, find the `KLAVIYO` object near the top of the `<script>`,
and fill in two values:

| key | where to find it |
| --- | --- |
| `publicKey` | Klaviyo → Settings → API keys → **Public API key** (6 characters) |
| `listId` | Klaviyo → Audience → Lists & Segments → open the list → Settings (6 characters) |

Both are safe in the browser: the public key identifies the account and grants
nothing beyond subscribing. **Never put a private key (`pk_...`) here** — this
integration does not need one, and there is no server.

Until both are set the form refuses to submit and says so, rather than silently
dropping addresses.

Signups POST to Klaviyo's `/client/subscriptions/` endpoint, the one intended to be
called from a browser. Each carries `custom_source: "Waitlist page"`, so waitlist
profiles can be segmented without a separate list. If the list has double opt-in
enabled, Klaviyo sends the confirmation email and the profile only joins on confirm.

Test it by submitting your own address on the deployed site and checking the list.

## Changing things

**Copy** — the `<h1>`, the `.lede` under it, the header wordmark and the `Early
access` badge are all plain text in the markup. Update `<title>`, `og:title` and the
meta description to match, or link previews will keep quoting the old headline.

**The photo** — replace `public/monitors.webp` and `public/monitors-800.webp`,
keeping the names. Use a transparent background (the page sits it directly on the
orange floor) and update `width`/`height` on the `<img>` to the real pixel size so
the layout does not shift as it loads. If you only have one size, point both the
`src` and `srcset` at it.

**Colour** — the palette is CSS custom properties on `:root` at the top of the
`<style>` block. `--ember`, `--violet` and `--ice` drive the field's gradient border
and its two glows; the floor is the `.glow` rule just below.

## Layout notes

Worth knowing before editing the CSS, since each one is load-bearing:

- `overflow-x: clip` lives on `html`, not on a flex wrapper. Putting it on `.shell`
  turns the y-axis into a scroll container and truncates the page.
- The field's `1px` gradient border is `padding: 1px` on `.field` over a solid
  `.field-inner`, not a `border`.
- The glows and the glyph texture are sized in percentages of the field's height,
  so changing the field's padding changes their reach.
- On desktop the art column deliberately runs off the right edge of the viewport;
  the photo's height is capped at `66vh` so it never forces the page to scroll.

Checked at 390, 768, 1024, 1280, 1440, 1920 and 2560 px wide.

## Fonts

`fonts.css` embeds **BR Omega**, a commercial typeface. It is included here so the
page looks exactly as designed, but confirm your licence covers this project before
shipping it publicly.

To switch to a free face instead: delete `fonts.css` and its `<link>`, then drop
`'BROmega'` from the `--sans` variable in `:root`. The page falls back to Plus
Jakarta Sans, already loaded from Google Fonts, and nothing else needs to change.
