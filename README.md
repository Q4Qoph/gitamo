# GMS / ARC

Astro static website for Gitamo Microelectronics Solutions. The site preserves its editorial layout and hash-routed pages, uses the supplied GMS logo assets, and applies the navy, blue, gold, and white brand palette.

## Stack

- Astro 7, static output
- HTML, CSS, and browser JavaScript
- Cloudflare Workers Static Assets as the primary hosting target
- Cloudflare Pages as a static-hosting alternative

No Astro Cloudflare adapter is needed: this project has no Astro API routes, SSR pages, or server-side form handler. Astro builds the site to `dist/`.

## Local development

Requirements: Node.js 22.12 or later and npm. The lockfile is `package-lock.json`.

```sh
npm ci
npm run dev
npm run build
npm run preview
```

`npm run check` invokes `astro check`, which also requires `@astrojs/check` and TypeScript. Those packages are not currently in the lockfile; the command prompts to install them. No lint or test scripts are configured.

## Cloudflare Workers

`wrangler.jsonc` points Workers Static Assets at `dist/` and uses the SPA fallback because current page navigation uses URL hashes. The `cf:preview` and `cf:deploy` scripts build first and invoke a pinned Wrangler 4 CLI. `cf:deploy` is an explicit production deployment command; it is never called by install, build, or CI.

```sh
npm run cf:preview
npm run cf:deploy
```

The repository is not connected to Cloudflare yet and this project has not been deployed. See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) before connecting it.

## Cloudflare Pages alternative

Use Framework preset **Astro**, production branch `main`, build command `npm run build`, and output directory `dist`. Pages is suitable for this static build; do not add the Cloudflare adapter unless server-side routes are introduced.

## Environment and secrets

No environment variables are required by the current site. The inquiry form creates a local text brief; it does not send submissions. Do not add credentials to `.env`, `.dev.vars`, source files, or GitHub.

## SEO and known limitations

- The app emits one static `/` document; internal navigation is hash-routed, so product and company views are not separate crawlable routes.
- Open Graph title and description are set. There is no canonical URL, sitemap, or production domain configured.
- No favicon was supplied as a standalone symbol asset, so none is configured.
- Contact details remain placeholders and the form has no backend. Do not use it for sensitive engineering data.
- The original prototype in `reference/` is internal and excluded from Git.
- No GitHub Actions deployment or Cloudflare automatic deployment is enabled. CI runs the locked production build. The separate Astro check command is not in CI until its missing dependencies are added to the lockfile.

## Repository

Target GitHub repository: <https://github.com/Q4Qoph/gitamo>
