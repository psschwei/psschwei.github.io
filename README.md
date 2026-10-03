# psschwei.github.io

Source for my personal website, [psschwei.com](https://psschwei.com/).

Built with [Hugo](https://gohugo.io/) and the [Hugo Flex](https://themes.gohugo.io/themes/hugo-flex/) theme. Pushes to `main` trigger a GitHub Actions workflow (`.github/workflows/deploy.yaml`) that builds the site and deploys it to GitHub Pages.

## Layout

- `content/` – pages and blog posts (Markdown)
- `static/` – files copied as-is to the site
- `layouts/` – template overrides for the theme
- `themes/hugo-flex/` – the theme
- `notes/` – backup copies of talk abstracts (not published on the site)
- `config.yaml` – site config and nav menu

## Local development

```sh
hugo server
```
