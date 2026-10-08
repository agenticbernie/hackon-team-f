# AGENTS.md

Notes for anyone (human or agent) working in this repo.

## What this is

The public website for **HackOn Team** — a single-page Astro site. Astro renders
pages to HTML; there is no server-side API, database, or external service, so the
app needs **no credentials** and no infrastructure services.

Positioning, copy rules and the official tagline are fixed by the brief: HackOn is
an *independent builder organization*, not a startup or an AI company. Do not add
funding, customer, user, revenue, or traction claims, and do not invent social
accounts. Bernie Nguyen is the only public contact (`bernie@hackon.team`,
LinkedIn `bernieweb3`).

## Structure

```
src/layouts/Layout.astro        head/meta + reveal & nav scripts (inline)
src/components/Nav.astro        sticky nav, mobile panel, active-section state
src/components/SectionHeader.astro
src/components/Hero.astro
src/components/ProjectCard.astro      (media via <slot name="media">)
src/components/AchievementCard.astro
src/components/EcosystemCard.astro
src/components/TeamCard.astro
src/components/CTABlock.astro
src/components/Footer.astro
src/components/sections/*.astro       one file per page section
src/pages/index.astro                 composes the sections
src/styles/global.css                 design tokens + shared primitives
```

Section headings live in each section component; edit the data arrays at the top
of `src/pages/index.astro`'s child components to change content.

## Assets

`public/brand/`, `public/team/`, `public/media/`. All source images are 1:1.
`logo-mark.png` is the transparent symbol used for nav/hero/footer;
`logo-full.png` (opaque, on near-black) is the OG image.

The AeroTwin AI demo is embedded from YouTube (`https://www.youtube.com/embed/K-CrOEKbgHQ`)
as a lazy-loaded iframe, so no large video file is shipped in `public/media/`.

## Running it in the Base44 sandbox

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

- The `web` service runs `node:22`, bind-mounts the repo at `/app`, installs
  dependencies into a named `node_modules` volume on startup, and runs
  `astro dev` with hot reload.
- The app is served on host port **3000** (the Astro dev server listens on
  `0.0.0.0:3000`, configured in `astro.config.mjs`).
- Healthcheck hits `http://localhost:3000/` inside the container.

Logs: `docker compose -f docker-compose.base44.yml logs -f web`

## Sandbox-only overrides

Both live in `astro.config.mjs` and are gated on `BASE44_PREVIEW_MODE === "1"`:

- **Vite host allowlist.** The preview proxy reaches the dev server as
  `Host: <port>-<sandbox id>.$BASE44_SANDBOX_HOST_DOMAIN`, which Vite rejects by
  default. In preview mode the allowlist is extended with `.$BASE44_SANDBOX_HOST_DOMAIN`.
  When the flag is unset or any other value, `allowedHosts` stays undefined and
  Astro keeps its default behavior. (The platform also sets
  `__VITE_ADDITIONAL_SERVER_ALLOWED_HOSTS`, passed through in compose.)
- Nothing else is sandbox-specific. `vite.server.watch.usePolling` is enabled
  unconditionally because bind mounts need it in containers; it has no effect on
  a normal local checkout.

## Verifying a change

```bash
curl -sS http://localhost:3000/ | head
```

Frontend edits hot-reload automatically. A change to `astro.config.mjs`,
`package.json`, or the compose file needs a service restart:

```bash
docker compose -f docker-compose.base44.yml restart web
```

Worth checking after edits: no console errors, every `[data-reveal]` block ends up
with `is-visible`, nav `is-active` follows the section in view, and the mobile
menu toggle (`[data-nav-toggle]`) opens/closes at <=900px.
