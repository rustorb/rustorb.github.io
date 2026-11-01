# rustorb.github.io

The source of the rustorb documentation site, <https://rustorb.github.io/>.
rustorb itself lives in [rustorb/rustorb](https://github.com/rustorb/rustorb).

The site is built with [MkDocs](https://www.mkdocs.org/) and
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). The
pages are Markdown files in [docs/](docs/): English in `docs/en/`, and
Japanese in `docs/ja/` with the same file names, published under `/ja/` by
[mkdocs-static-i18n](https://github.com/ultrabug/mkdocs-static-i18n). A
page missing in Japanese falls back to the English one. The menu is `nav`
in [mkdocs.yml](mkdocs.yml); its Japanese titles are `nav_translations`
there.

```sh
pip install -r requirements.txt
mkdocs serve                 # http://127.0.0.1:8000/, reloads on save
mkdocs build --strict        # what CI runs; the result is in site/
```

Every push to `main` builds the site and publishes it with GitHub Pages
([.github/workflows/pages.yml](.github/workflows/pages.yml)); pull requests
only build it. The repository's Settings → Pages → Build and deployment →
Source must be **GitHub Actions**.

The documentation is licensed under MIT OR Apache-2.0, like rustorb.
