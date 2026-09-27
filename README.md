# visionops.cz

Hugo site deployed to GitHub Pages by `.github/workflows/hugo.yml`.

## New content

```sh
hugo new content blog/<slug>.md      # archetypes/blog.md
hugo new content sluzby/<slug>.md    # archetypes/sluzby.md
```

Fill in `title` (max ~50 chars), `description`, `breadcrumb`, `keywords`, then remove `draft: true`.

## Pipeline

Every push and pull request:

1. `hugo --panicOnWarning`
2. [lychee](https://github.com/lycheeverse/lychee-action) checks internal links
3. [Lighthouse CI](https://github.com/treosh/lighthouse-ci-action) requires SEO score 100 on every page (`lighthouserc.json`)

On `master` the site is deployed and pages changed in the last day are submitted via
[IndexNow](https://github.com/bojieyang/indexnow-action) (Bing, Seznam, Yandex, DuckDuckGo via Bing).
Google reads `sitemap.xml` from `robots.txt`; indexing status is in Search Console.

Local check:

```sh
hugo --gc --minify --panicOnWarning && npx @lhci/cli autorun
```
