Personal website for [https://williammora.com](https://williammora.com), powered by the GitHub Pages-supported [Jekyll](https://jekyllrb.com) stack.

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
