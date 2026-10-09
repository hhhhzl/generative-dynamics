# GenerativeDynamics website

A static research and framework documentation website. This first edition includes the MDOC → MD-COAS → 2GO → MGA research trajectory, an extensible method catalog, original demonstrations, and framework documentation.

## Build and preview

Requires Node.js 20 or newer and Python 3 for the optional local server.

```sh
npm install
npm run build
python3 -m http.server 8767 --bind 127.0.0.1 --directory dist
```

Open http://127.0.0.1:8767/. The generated `dist/` directory can be served by a static web host. It is checked in for reproducible deployment. The build can also use an existing Marked installation with `GD_MARKED_PATH` set to its ESM file URL.

## Extend the website

The current four papers are entries, not a fixed site limit. Research detail pages, story tabs, publication cards, and the sitemap are generated from their registry.

| Content | Edit |
| --- | --- |
| Framework name, repository, version, canonical origin | `content/site.json` |
| Research papers, their story, evidence, links, and associated demos | `content/research.json` |
| Method catalog, including methods without a dedicated paper | `content/methods.json` |
| Homepage CPU, GPU, and Docker code examples | `content/examples.json` |
| Documentation navigation and grouping | `content/docs.json` |
| Documentation articles | `content/docs/*.md` |
| Demo metadata, captions, playback qualifications, and homepage selection | `content/media.json` |
| Original video and diagram files | `assets/media/` |
| Page templates | `build.mjs` |
| Visual system and responsive layouts | `assets/style.css` |
| Menus, research tabs, demo filters, video playback, and documentation search | `assets/app.js` |

To add a paper, copy a complete entry in `content/research.json`, assign a unique URL-safe `id`, and update its text, publication metadata, results, limitations, media IDs, and recipe. Add a corresponding `content/methods.json` record with `research` set to that ID. A method without a paper only needs a method-catalog record. Keep research entries ordered to express the intended story.

To add a documentation page, save its Markdown in `content/docs/` and register its path, slug, title, and group in `content/docs.json`. Use site-relative links to `/docs/<slug>/`. Documentation search filters page titles in the sidebar; it is not a full-text search engine.

To add a demonstration, place the video in `assets/media/`, add its metadata to `content/media.json`, and reference its ID in a paper, `homepageIds`, or `heroIds`. The hero selection requires hardware clips. MP4 clips are used in place of GIFs to reduce transfer size and provide controls. Preserve the source, hardware/simulation distinction, and playback-speed qualification. 2GO and hardware clips use `timingPolicy: "source"` and `playbackRate: 1`; retain original timestamps when re-encoding and record the selected source interval. Never shorten a clip by speeding it up.

After changing content, rebuild. Each build refreshes the generated `dist/` directory. Keep internal links and citation targets valid.

## Content provenance

- Framework documentation was adapted from the supplied GenerativeDynamics README, framework implementation, and existing documentation, reviewed on 2026-10-09.
- Research descriptions and numerical results were grounded in the supplied LaTeX manuscripts for MDOC, MD-COAS, 2GO, and MGA. Each detail page includes scope and experimental qualifications.
- Demonstrations are the supplied original research clips referenced from the framework's media materials and additional submissions. Presentation playback may differ from execution speed.
- The release is described as Alpha. The repository may require access while the release is prepared. Simulator, accelerator, and learning support should be checked on the Compatibility page.
- MGA remains labeled as a research manuscript because no verified public paper URL was supplied.

## Hosting

`.openai/hosting.json` points Sites to `dist/` and preserves this site's project identity. Publish the exact built source state and its deployment archive together. This first edition is owner-private.

The static output can also be deployed to another provider. `SITE_ORIGIN` overrides the canonical host at build time; `SITE_BASE_PATH` prefixes local links and assets for a project subdirectory. Leave the base path empty for a root-domain site.

## GitHub Pages deployment

The included `.github/workflows/deploy-pages.yml` builds with Node.js 24 and publishes only `dist/`. It reads the actual origin and base path from GitHub Pages, so both a project site and a user site work without editing content or templates.

1. Create an empty GitHub repository, for example `hhhhzl/generative-dynamics`. A public repository works with GitHub Free. Do not initialize it with a README if you will push this checkout's existing history.
2. Push this website checkout to that repository's `main` branch. The framework repository is separate; these instructions publish the website source.

```sh
cd /Users/zhilinhe/Documents/Codex/2026-10-08/ca/outputs/generative-dynamics
git remote add github https://github.com/hhhhzl/generative-dynamics.git
git push -u github HEAD:main
```

3. In the GitHub repository, open **Settings → Pages → Build and deployment → Source**, and select **GitHub Actions**.
4. Open **Actions → Deploy website to GitHub Pages → Run workflow → main**. The first push may have run before Pages was enabled; this manual run publishes after setup. Once both jobs succeed, GitHub reports the live URL. For the example project repository, the default URL is `https://hhhhzl.github.io/generative-dynamics/`.
5. Later pushes to `main` rebuild and publish automatically. If the repository uses another branch, update the workflow's `push.branches` and GitHub environment rules accordingly.

For `https://hhhhzl.github.io/`, use a repository named `hhhhzl.github.io`; each account has one user site. An independent project repository is convenient when that user site is already in use. If `github` is already configured as a remote, use its existing configuration or update its URL instead of adding it again.

To inspect a GitHub-style project build locally:

```sh
SITE_ORIGIN=https://hhhhzl.github.io SITE_BASE_PATH=/generative-dynamics npm run build
```

Serve `dist/` under `/generative-dynamics/` when previewing that build; serving it at `/` will not emulate GitHub's project path. Run `npm run build` without these environment variables before previewing or publishing on the original root-domain host.

GitHub Pages setup documentation: [publishing source](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site), [custom workflows](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

## Validation

The first edition was checked for missing local assets, internal routes, anchor targets, duplicate IDs, JavaScript syntax, and representative desktop/mobile layouts. Research tabs, result filters, documentation navigation, and page-title filtering were exercised in the browser. These website checks do not reproduce the robotics experiments.
