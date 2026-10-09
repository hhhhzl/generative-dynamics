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
| Documentation navigation and grouping | `content/docs.json` |
| Documentation articles | `content/docs/*.md` |
| Demo metadata, captions, playback qualifications, and homepage selection | `content/media.json` |
| Original video and diagram files | `assets/media/` |
| Page templates | `build.mjs` |
| Visual system and responsive layouts | `assets/style.css` |
| Menus, research tabs, demo filters, video playback, and documentation search | `assets/app.js` |

To add a paper, copy a complete entry in `content/research.json`, assign a unique URL-safe `id`, and update its text, publication metadata, results, limitations, media IDs, and recipe. Add a corresponding `content/methods.json` record with `research` set to that ID. A method without a paper only needs a method-catalog record. Keep research entries ordered to express the intended story.

To add a documentation page, save its Markdown in `content/docs/` and register its path, slug, title, and group in `content/docs.json`. Use site-relative links to `/docs/<slug>/`. Documentation search filters page titles in the sidebar; it is not a full-text search engine.

To add a demonstration, place the video in `assets/media/`, add its metadata to `content/media.json`, and reference its ID in a paper or `homepageIds`. MP4 clips are used in place of GIFs to reduce transfer size and provide controls. Preserve the source, hardware/simulation distinction, and playback-speed qualification.

After changing content, rebuild. When removing a route, also remove its old directory from `dist/` before rebuilding; the generator does not erase stale routes. Keep internal links and citation targets valid.

## Content provenance

- Framework documentation was adapted from the supplied GenerativeDynamics README, framework implementation, and existing documentation, reviewed on 2026-10-09.
- Research descriptions and numerical results were grounded in the supplied LaTeX manuscripts for MDOC, MD-COAS, 2GO, and MGA. Each detail page includes scope and experimental qualifications.
- Demonstrations are the supplied original research clips referenced from the framework's media materials and additional submissions. Presentation playback may differ from execution speed.
- The release is described as Alpha. The repository may require access while the release is prepared. Simulator, accelerator, and learning support should be checked on the Compatibility page.
- MGA remains labeled as a research manuscript because no verified public paper URL was supplied.

## Hosting

`.openai/hosting.json` points Sites to `dist/` and preserves this site's project identity. Publish the exact built source state and its deployment archive together. This first edition is owner-private.

The static output can also be deployed to another provider. For a different domain, update `content/site.json` → `origin` and rebuild. A subdirectory deployment requires adapting absolute site-relative links and assets.

## Validation

The first edition was checked for missing local assets, internal routes, anchor targets, duplicate IDs, JavaScript syntax, and representative desktop/mobile layouts. Research tabs, result filters, documentation navigation, and page-title filtering were exercised in the browser. These website checks do not reproduce the robotics experiments.
