## zackkoppert.com

Personal blog and portfolio for Zack Koppert - Senior Software Engineering Manager at GitHub.

### Tech stack

- [Jekyll](https://jekyllrb.com/) with the [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme
- Hosted on [GitHub Pages](https://pages.github.com/) with a custom domain
- Deploys automatically on push to `main` via GitHub Actions

### Local development

```bash
bundle install
bundle exec jekyll serve
```

Then open [http://localhost:4000](http://localhost:4000).

### Deploy

Pushes to `main` trigger the `pages-deploy.yml` workflow, which builds the site and deploys to the `gh-pages` branch. Pull requests get automatic preview deploys via `pr-preview.yml`.

### Attribution

- [Chirpy Jekyll Theme](https://github.com/cotes2020/jekyll-theme-chirpy) by Cotes Chung (MIT License)
- Originally based on the [Minimal](https://github.com/pages-themes/minimal) theme and [evanca/quick-portfolio](https://github.com/evanca/quick-portfolio)
