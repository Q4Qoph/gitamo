# Deployment preparation

The current app is a static Astro build. It has one generated `/` document and uses client-side hash routing for its existing views. It has no server endpoints, SSR pages, database bindings, or live contact submission. Use Workers Static Assets without `@astrojs/cloudflare`; add the Astro adapter only if server-rendered routes are later approved and implemented.

## Current configuration

- Worker name: `gitamo`
- Compatibility date: `2026-11-02` (review/update on the deployment date)
- Static asset directory: `./dist`
- Unknown paths: SPA fallback to `index.html` for the existing hash router
- Worker entry point: none
- Environment variables/secrets: none required
- Production domain/account IDs: intentionally unset

Wrangler scripts are explicit and do not deploy during install, development, CI, or ordinary builds:

```sh
npm run cf:preview
npm run cf:deploy
```

The preview script builds the static site and starts Wrangler locally. The deploy script builds and publishes to Workers; do not run it until the production checklist below has been approved.

## Connect the GitHub repository to Workers

1. Confirm the public source repository is approved for publication and contains no `.env`, `.dev.vars*`, access tokens, or internal `reference/` files. The repository target is <https://github.com/Q4Qoph/gitamo>.
2. In the Cloudflare dashboard, open **Workers & Pages** and choose **Create application** / **Import a repository**. Connect GitHub and select `Q4Qoph/gitamo`.
3. Choose **Workers Builds** / a Worker application, not a Pages project. Keep the Worker name `gitamo` unless the owner chooses another name.
4. Use the repository's Wrangler configuration and set the build command to `npm run build`. For the deploy command, use `npx wrangler@4.156.1 deploy`. Wrangler reads `wrangler.jsonc` and uploads `dist/` as static assets. Do not add an Astro adapter or a Worker entry point for this static site.
5. Keep automatic production deployments disabled during initial setup. First build and deploy a preview/staging version, review its URL, and validate the pages and asset paths.

See Cloudflare's [Astro on Workers guide](https://developers.cloudflare.com/workers/framework-guides/web-apps/astro/), [Astro deployment guide](https://docs.astro.build/en/guides/deploy/cloudflare/), [Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/), and [Wrangler configuration reference](https://developers.cloudflare.com/workers/wrangler/configuration/).

## Configure secrets and environment variables

The current `public/scripts/legacy.js` config has an empty inquiry endpoint and disabled submission/privacy gates. The form downloads a local text brief and sends no network request. No environment variable or Cloudflare secret is required today.

If a real backend is approved later, define and document the exact variable names first. Add secret values in the Cloudflare dashboard or with `wrangler secret put NAME`; never commit actual values. Do not put credentials in GitHub Actions, `.env`, `.dev.vars`, or `wrangler.jsonc`. The current form must not be represented as a live submission endpoint.

## Preview and verify

Local Workers runtime preview:

```sh
npm ci
npm run cf:preview
```

For a dashboard-managed staging deployment, use a non-production Worker/environment and an approved preview URL. Confirm:

- Home page, hash navigation, product views, and the footer load.
- Both `/brand/` WebP logos and generated CSS/JS assets return successfully.
- The mobile menu and hero controls work at desktop and narrow widths.
- The inquiry form downloads its local draft and does not claim to submit it.
- The console is free of new errors and no local-only URL appears in the built output.

Do not use a production custom domain for the initial preview.

## Custom domain, HTTPS, and production routing

1. Obtain the client's approved domain and DNS instructions. No domain is configured in this repository.
2. Add the domain in the Worker’s **Domains & Routes** settings only after explicit approval. Do not change DNS from this repository task.
3. Wait for Cloudflare to validate the domain and issue HTTPS. Confirm the certificate is active before announcing the URL.
4. Verify `/`, `/brand/gms-logo-header-transparent.webp`, `/brand/gms-logo-full-transparent.webp`, and the Workers SPA fallback over HTTPS.
5. Review `robots.txt`, canonical metadata, Open Graph image, sitemap, favicon, privacy notice, and contact details once a production domain and launch content are approved. No canonical URL or sitemap is generated now because the domain is unknown and the site has one hash-routed HTML document.

Workers Static Assets currently attaches `Content-Type`, an ETag, and a revalidation-oriented cache header by default; hashed assets can be cached efficiently. Review Cloudflare's [static asset headers](https://developers.cloudflare.com/workers/static-assets/headers/) before adding custom `_headers` rules.

## Enable automatic deployments after approval

After the staging review and production approval:

1. Select `main` as the production branch in Workers Builds.
2. Enable production deployment on pushes to `main`.
3. Keep pull-request builds as previews where the project settings support them.
4. Confirm CI is green before merging. The current GitHub Action runs the locked Astro build only; this project has no lint/test scripts, and `astro check` requires `@astrojs/check` and TypeScript, which are not currently in the lockfile.
5. Make the first production deployment from an approved commit and verify its deployment ID and URL in Cloudflare.

Do not add Cloudflare deployment credentials to the GitHub Actions workflow unless the owner later chooses an explicitly reviewed CLI-based deployment pipeline. This workflow does not deploy.

## Roll back

If a release fails, use the Cloudflare dashboard's Workers deployment/version history to restore the last known-good version. Then revert the bad Git commit and confirm CI before creating a replacement deployment. Do not rewrite Git history or force-push.

## Cloudflare Pages alternative

Pages is compatible with this static build and does not require `@astrojs/cloudflare`. In the Pages dashboard choose **Astro**, then configure:

| Setting | Value |
| --- | --- |
| Production branch | `main` |
| Build command | `npm run build` |
| Build output directory | `dist` |

Pages handles static assets and its Git integration can create previews for pull requests. Do not configure Workers and Pages as simultaneous production targets. If SSR or API routes are added later, reassess the Pages Functions/adapter compatibility separately rather than assuming this static configuration supports them.

Official references: [Cloudflare Pages Astro guide](https://developers.cloudflare.com/pages/framework-guides/deploy-an-astro-site/) and [Pages build configuration](https://developers.cloudflare.com/pages/configuration/build-configuration/).

## Production checklist

- [ ] Client confirms public repository visibility and approves the copy/assets for public distribution.
- [ ] Internal `reference/` material and all secrets remain excluded from Git.
- [ ] CI build passes on the commit selected for release.
- [ ] Staging preview reviewed on desktop and mobile.
- [ ] Form backend, privacy notice, and safe data handling are approved before enabling submissions.
- [ ] Official public contact details and custom domain are approved.
- [ ] Canonical URL, sitemap, robots policy, Open Graph image, and favicon are reviewed for the approved domain.
- [ ] Cloudflare HTTPS and routing are verified.
- [ ] Owner explicitly approves enabling automatic production deployments.
