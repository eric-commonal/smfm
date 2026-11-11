# SMFM Novi — custom styles and content blocks

Custom CSS and HTML blocks for SMFM's Novi site (`smfm.novistaging.com`), which
becomes the Member Portal. Served to Novi via GitHub Pages.

**This repo is the source of truth for the CSS.** Do not also keep a copy in
Novi's Custom CSS field — two copies drift, and Novi's is the one nobody
remembers to update.

## Files

| File | What it is |
|---|---|
| `smfm-tokens.css` | Design tokens plus a narrow set of global typography overrides. Loads first. |
| `smfm-blocks.css` | The custom block components. Loads second, depends on the tokens. |
| `smfm-novi-head-code.html` | Reference copy of what goes in Novi's head-code field: font links, the two stylesheet links, and the sitewide scripts. |
| `smfm-block-hero.html` | Hero block markup, to paste into a Novi page content editor. |
| `smfm-style-review-page.html` | Page content for an internal `/style-review` page: every block with a Copy Code button. Restrict it with Novi's page visibility settings. |
| `smfm-blocks-preview.html` | Local preview harness. Open it in a browser directly. |

## Deployment

1. The two stylesheets are served from GitHub Pages:
   - `https://eric-commonal.github.io/smfm/smfm-tokens.css`
   - `https://eric-commonal.github.io/smfm/smfm-blocks.css`
2. Novi's head-code field carries the contents of `smfm-novi-head-code.html`.
   That field is pasted once; after that, pushing to this repo updates the live
   site with no Novi change.
3. Block markup is pasted per page into Novi's page content editor.

Expect a minute or two of cache lag between a push and the live site. Hard
refresh before assuming a change failed.

## Where the design comes from

Every value in `smfm-tokens.css` was derived from computed styles on
`test-smfm.pantheonsite.io` (the Drupal rebuild), measured 2026-11-11. That
site ships no CSS custom properties, so nothing could be copied directly.

The block vocabulary deliberately mirrors the Drupal component system so one
mental model covers both properties. The mapping table is at the top of
`smfm-blocks.css`.

## Things that will bite you

- **Novi strips `<script>` from pasted page content.** All JavaScript lives in
  the head-code field. Block markup never carries its own script.
- **Novi deletes any element with an `onclick` attribute** — the whole element,
  not just the attribute. Use a class plus a `data-*` attribute and a delegated
  handler.
- **Novi does not preserve `<div style="display:contents">` wrappers.** They
  collapse into empty `<p>` tags and content gets reordered. Never rely on
  wrapper divs or sibling adjacency; match by `data-*` attribute.
- **Nesting depth varies by template.** The homepage puts `.body-content`
  directly under `<body>` at full width; interior pages nest through
  `.container > .row > .o-column` and inset content. Do not write selectors
  that depend on exact nesting or `:first-child`.
- **Full-bleed sections are positioned by script, not CSS.** Anything carrying
  `smfm-bleed` gets inline `margin-left` / `margin-right` / `width` from the
  head-code script. Never declare those properties on those elements in CSS,
  and never override the inline values. Inner content re-centres via
  `.smfm-wrap`.
- **The global heading rules in `smfm-tokens.css` are `!important`.** Any
  dark-surface text override has to out-specify them *and* explicitly exempt
  anything that is not white. See section 9 of `smfm-blocks.css`.
- **Do not add a global heading `color` override.** It was tried and reverted.
  Novi has stock components with dark backgrounds (`.widget` and friends) whose
  headings Novi correctly sets to white; an `!important` colour override
  repaints those to purple and drops contrast to 1.55:1. Dropping `!important`
  doesn't help — Novi's heading selectors are more specific, so the override
  then loses everywhere. Use the per-page or `.smfm-ink-headings` opt-in
  instead, and check the page for dark panels before opting in.
- **Per-page CSS** is available through the body class derived from the page
  slug (`.c-body-content__<slug>`). Verify the real class in view-source first.

## Buttons

`smfm-tokens.css` restyles Novi's own `.btn-primary` and `.btn-secondary`
sitewide, including on checkout, registration, profile and login. This is the
one deliberate exception to the typography-only rule for global overrides.

The mapping follows how Novi *uses* those classes rather than what they are
named: Novi puts `.btn-secondary` on primary conversion actions such as "Join
as Regular (MFM)", so `.btn-secondary` becomes the accent-fill button and
`.btn-primary` becomes the dark-fill one. Both keep their existing weight
relationship.

`.search-go` and `.log-out-btn` are excluded from the geometry changes —
they are a search icon and profile-menu rows, and button padding collapses
them. They still take the new colours.

Button colours for both Novi's `.btn` and the custom `.smfm-btn` come from the
same tokens, so the two systems cannot drift.

## Known open items

- Button label is purple on teal (6.14:1), not Drupal's white on teal (2.76:1).
  Change `--smfm-btn-primary-fg` to `var(--smfm-white)` for Drupal-exact; both
  button systems follow, and contrast drops below AA sitewide.
- Novi's logged-in system pages (checkout, profile, event registration, member
  compass) have not been inspected. Global overrides are kept to typography
  partly for that reason.
- Hero image is a placeholder. Needs a square source, 680x680 or larger,
  uploaded to Novi file uploads.
