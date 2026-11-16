# Independent Markdown blog plan

## Goal

Publish new articles by committing Markdown without rebuilding or redeploying the ARM64 LabFabric application. Use `blog.labfabric.pro` for the new site. Keep the existing `/learn/...` and `/ccna/...` guides at their current URLs.

## Local prototype

The `blog/` directory is a working Hugo site. Its home page mirrors the current blog's layout, colors, article cards and footer. Five existing guides remain linked to the main site through `blog/data/existing_guides.yaml`. A draft Markdown article in `blog/content/` exercises the article template without appearing in production.

Install Hugo 0.170.0, then run these commands **one at a time**:

```sh
hugo --source blog --buildDrafts --destination /tmp/labfabric-hugo-preview
hugo server --source blog --buildDrafts --bind 0.0.0.0 --port 1313
```

Before sharing a preview URL, check the host's current LAN IP. Stop the server before running another test or build. For a production-equivalent local build, omit `--buildDrafts`; the example article will be excluded.

Tested locally on 16 November 2026 with Hugo 0.170.0: preview and production builds completed without warnings; the draft appears only in the preview; the home page, draft article, CSS and cover image returned HTTP 200. Desktop (1440 px) and phone (390 px) browser screenshots showed the same overall article-card layout, colors and spacing as the current blog. The article template rendered its Markdown headings and code blocks. The navigation and dark mode are visually close but simpler than the interactive Next.js versions (superseded by the header update below).

### Header parity update (16 November 2026)

The Hugo header now copies the live production header rather than approximating it. Verified against https://labfabric.pro/blog with Playwright screenshots (dark scheme, 1440 px): same logo plate, `Learn` and `Practice` dropdowns with the live menu entries, Blog, Pricing, Contact, Availability, My Account, Book a Demo and Start free lab, in the same positions and sizes. The logo uses the real `.lf-logo` rules from `src/app/globals.css` (mark stroke-draw, wordmark print wipe, caret boot pause then ten blinks, reduced-motion handling). Every link is an absolute `https://labfabric.pro/...` URL except Blog, which points to the blog home. The menus are CSS-only (`<details name>`), the phone layout collapses into a checkbox hamburger below 1100 px, and Inter Tight is self-hosted. The header is in `blog/layouts/_default/baseof.html` and the styles are appended to `blog/static/css/blog.css`.

Follow-up changes (same day):
- **Logo is pixel-checked against prod.** It uses prod's real markup (the lucide Terminal SVG paths, 24px viewBox shown at 20px, 12px plate radius from `rounded-lg`) and the Inter Tight woff2 files copied from labfabric.pro into `blog/static/fonts/` (latin and latin-ext, same `@font-face` rules and fallback), so the font loads the same way. A 2x Playwright diff of the logo area against https://labfabric.pro/blog (dark, 1440 px, after the animation finishes, and again with reduced motion) shows the same 141.6 x 34 box, and no pixel differs by more than 8/255. Inside the plate the maximum difference is 4/255 (page-background blur behind the translucent plate), so it is not strictly bit-identical. A mid-animation frame was not compared.
- **Theme toggle** copies prod: `<html class="dark">`, `localStorage` keys `theme` and `theme-manual`, light 07:00 to 19:00 and dark otherwise on the visitor's clock until they click, same Sun/Moon icons and button. An inline script in `<head>` sets it before first paint. The `prefers-color-scheme` logic is gone.
- **Sharing the preference.** `localStorage` is per origin, so `blog.labfabric.pro` cannot see `labfabric.pro`'s choice. On a click the blog also writes a cookie `lf-theme=light|dark` on `.labfabric.pro` and reads it when no manual choice exists. The main site does not read that cookie yet. To share fully, the main site's pre-paint script in `RootShell.tsx` must read `lf-theme` too and its toggle must write it. That is a `src/` change and was not made.
- **My Account.** The header always shows "My Account", linking to `https://labfabric.pro/portal`. The main site's middleware redirects signed-out visitors from `/portal` to `/login`, so no session detection, cookie or CORS work is needed on the blog.
- Dropdown groups (CCNA, CCNP Enterprise, Guides) are collapsed with chevrons, and every entry has its lucide icon in cyan.

Remaining gaps:
- The menu is copied from the deployed site, not the uncommitted `Navbar.tsx` (which differs). Copy the header again when the main nav changes; a shared JSON of nav links would avoid that.
- Dropdown description text wraps slightly narrower than prod; the popup's exact width was matched by eye.
- Hover and focus states, the phone layout against prod, and a mid-animation logo frame were not compared.

## Cloudflare Pages rollout

1. Put `blog/` in a small, separate Git repository, or connect this repository to Pages with `blog` as the root directory. A separate repository keeps content edits out of the application release branch and makes the publishing workflow clear.
2. Create a Cloudflare Pages project linked to that repository. Use Hugo version `0.170.0` (`HUGO_VERSION=0.170.0`), build command `hugo`, and output directory `public`. If Pages uses this repository, set its root directory to `blog`. Set the same Hugo version for preview builds.
3. Add `blog.labfabric.pro` as a Pages custom domain and let Cloudflare create/verify its DNS record and TLS certificate. Check the preview domain and the production domain before changing main-site links.
4. Make one normal LabFabric release to change the main site's Blog navigation and `/blog` destination to `https://blog.labfabric.pro/`. Keep existing article URLs stable; add a permanent `/blog` redirect only after the new blog is live. Update the main sitemap to point to the new blog and avoid two competing blog index URLs. Hugo generates its own sitemap for new posts.
5. Publish by adding `blog/content/<slug>.md` with title, date, description, category, readingTime and image front matter, setting `draft: false`, and pushing to the Pages production branch. Confirm the Pages build, article URL, card, metadata, sitemap and mobile layout. Roll back a post by reverting its content commit in the blog repository; no app image changes.

Cloudflare references: [deploy a Hugo site](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/) and [add a custom domain](https://developers.cloudflare.com/pages/configuration/custom-domains/).

## Limits to account for

The Hugo template duplicates a small part of LabFabric's styling. Changes to the main navigation, theme or footer will need to be copied to the blog. Existing interactive guides should stay in Next.js because their diagrams and exercises are React components. The prototype keeps its cover images in `blog/static/images/`; new posts can use that directory too. Cloudflare Pages builds the small Hugo site for each content change; it does not trigger the app's Cloud Build or Docker release.
