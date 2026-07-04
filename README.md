# yakov.khalinsky.com

Personal site and research publications, served via GitHub Pages.

## URL

https://yakov.khalinsky.com

## Local preview

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000

## Structure

- `index.html` — refreshed personal landing page
- `styles.css` — site styles
- `agent-fleet-research/` — built research mini-site (MkDocs Material)
- `life/`, `dungeon/` — static experiments
- `CNAME` — custom domain config
- `.nojekyll` — bypass Jekyll processing

## Research mini-site

The research papers live at `/agent-fleet-research/` and are built from a separate source repo. To rebuild:

```bash
cd /home/yakov/agent-fleet-research-site
.venv/bin/mkdocs build --strict
cp -r site/* /home/yakov/yakovkhalinsky.github.io/agent-fleet-research/
```
