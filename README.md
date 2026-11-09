# DB Studio documentation

**Live site: <https://hasinishraq.github.io/db-studio-documentation/>**

The documentation site for DB Studio, built with [MkDocs](https://www.mkdocs.org/) and the
[Material](https://squidfunk.github.io/mkdocs-material/) theme.

```bash
cd documentation
pip install -r requirements.txt
mkdocs serve          # live preview at http://127.0.0.1:8000
mkdocs build --strict # static site in documentation/site/
```

Pages live in `docs/`. Screenshots in `docs/assets/screenshots/` are copies of the repository's
`screenshots/` folder; retake them with the scripts in `tests/e2e/screenshots/` and copy them
again.
