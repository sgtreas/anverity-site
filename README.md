# site/

The First Anvil marketing site. Static HTML/CSS — no build step, no
JavaScript, no server required. Lives in this folder so it travels
with the product code, but is **independent** from the app build:
deploying the site does not affect the app and vice versa.

## Preview locally

```bash
open site/index.html
```

That's it. Every page renders correctly from `file://` URLs. No need
for a web server.

If you want a friendlier preview that handles relative URLs more
like the real deploy will:

```bash
cd site && python3 -m http.server 8000
open http://localhost:8000
```

## File layout

```
site/
├── index.html            ← landing page
├── style.css             ← shared stylesheet
├── privacy.html          ← Privacy Policy (mirrors AboutAndLegal.swift)
├── acknowledgments.html  ← open-source credits
├── appcast.xml           ← Sparkle update feed (stub for now)
└── README.md             ← this file
```

## Deploying

You have three reasonable hosts. Pick whichever is least friction.

### Option A — GitHub Pages (free, easiest)

1. Create a new GitHub repo: `sgtreas/firstanvil-site` (PUBLIC — GitHub
   Pages only works free on public repos)
2. Push the contents of THIS `site/` directory to the root of that repo
3. In repo settings → Pages → Source → main branch / root
4. Your URL becomes `sgtreas.github.io/firstanvil-site`
5. (Optional) Custom domain: add `firstanvil.uvinto.com` to repo
   Pages settings and CNAME it from your uvinto.com DNS

### Option B — Cloudflare Pages (free, faster CDN)

1. Sign in to dash.cloudflare.com → Pages → Create application
2. Connect to the same GitHub repo you used for Option A
3. Cloudflare auto-deploys on every push
4. Add custom domain: `firstanvil.uvinto.com`
5. Cloudflare handles HTTPS automatically

### Option C — uvinto.com subdirectory

If uVinto already has a web host (whatever serves uvinto.com), just
upload the contents of this `site/` directory to a
`firstanvil/` subdirectory. URL becomes `uvinto.com/firstanvil`. Same
files, same result.

## After deploying

Two things to update in the app:

### 1. Sparkle feed URL

If you publish to a URL other than `https://firstanvil.uvinto.com/appcast.xml`,
update two places to match:

- `Resources/Info.plist` → `SUFeedURL`
- `Sources/FDApp/AboutAndLegal.swift` → `SparkleDelegate.feedURLString`

Commit, push, cut a new release tag — the next release embeds the
correct feed URL so future updates work.

### 2. Generate the Ed25519 signing keypair (one time)

This is the cryptographic basis of safe auto-updates. Once generated,
the public key embeds in the app and the private key signs every
release.

```bash
# From the project root, after a successful swift build:
.build/checkouts/Sparkle/bin/generate_keys
```

Output:
- Public key (base64) → paste into `Info.plist` as `SUPublicEDKey`
- Private key → automatically saved in macOS Keychain.
  Export and back up to 1Password / a secure password manager —
  if you lose it, you can never sign updates for existing installs.

Then for every release going forward:

```bash
# After GitHub Actions has produced FormaDiscovery-vX.Y.Z.zip:
.build/checkouts/Sparkle/bin/sign_update FormaDiscovery-vX.Y.Z.zip
```

Copy the resulting `sparkle:edSignature="..."` value and add an
`<item>` entry to `appcast.xml`. Push the updated appcast to the
hosting repo. Users get the update notification on their next daily
check.

## Editing copy

- **Voice rules**: see `../Docs/POSITIONING.md` — quiet authority,
  no "game-changing," declarative sentences with periods.
- **Color palette**: defined as CSS variables at the top of `style.css`.
  Midnight navy `#0F172A`, burnished bronze `#B8860B`, cream
  `#FAF6EB`. Edit there, applies everywhere.
- **Typography**: Bodoni Moda from Google Fonts for the wordmark and
  display headings; system serif/sans fallback for body. Single import
  at the top of `style.css`.

## What's intentionally NOT here

- **JavaScript**: none. The page works without it. Faster, more
  accessible, no analytics surface to manage.
- **External analytics**: no Google Analytics, no tracking pixels.
  Matches the privacy positioning of the product itself.
- **Email signup form**: when we want one, the simplest path is
  Buttondown or ConvertKit's embedded form snippet. Add to
  `index.html`'s download section.
- **Stripe checkout**: when we want to actually charge, swap the
  current "Download" button for a Stripe Checkout link or Buy button.
  Until then, downloads from the GitHub Release URL are free.

## Accessibility checklist (when you ship)

- [ ] Color contrast verified (cream/navy pair is ~13.5:1, well past WCAG AAA)
- [ ] Every `<img>` has alt text (only the SVG mark currently, has aria-label)
- [ ] Keyboard navigation works (all links + buttons, no focus traps)
- [ ] Page renders at 320px width (smallest iPhone landscape)
- [ ] Page renders at 2560px width without weird empty space
- [ ] `<title>` is meaningful per page
- [ ] Each page has one `<h1>`
- [ ] Meta description tag present and useful
