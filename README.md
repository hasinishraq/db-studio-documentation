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

Pages live in `docs/`. Screenshots in `docs/assets/screenshots/` are copies of the app
repository's `screenshots/` folder (171 shots, numbered in workflow order: `001`–`104` dark-theme
workflows, `105`–`146` settings, audit log, palette and assistant, `147`–`169` light theme).
Retake them with the scripts in `tests/e2e/screenshots/` and copy them again, keeping the file
names so the pages keep working.
