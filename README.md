Personal website for [https://williammora.com](https://williammora.com), powered by the GitHub Pages-supported [Jekyll](https://jekyllrb.com) stack.

## Posting

- Write: `bin/post "Title of the post"` (add `--draft` to start in `_drafts/`; `bin/publish` picks up a post started another way, as long as the only changes are under `_posts/` or `_drafts/`).
- Preview: `mise exec -- bundle exec jekyll serve --drafts`
- Publish: `bin/post ... --publish`, or `bin/publish` for a post already written

GitHub Pages rebuilds the live site within about a minute of a push to `master`. Posts do not need an `<!--more-->` separator — the index shows the whole post when one is missing.

This site pins Ruby 3.3.4 via [mise](https://mise.jdx.dev) (`mise.toml`); the
macOS system Ruby is too old to build the `github-pages` gem.

To run it locally:

```bash
mise install
mise exec -- bundle install
mise exec -- bundle exec jekyll serve
```

If Nokogiri fails to compile on macOS during `bundle install`, retry with:

```bash
NOKOGIRI_USE_SYSTEM_LIBRARIES=1 mise exec -- bundle install
```

Most posts in this repository are archival and intentionally kept online as originally published.
