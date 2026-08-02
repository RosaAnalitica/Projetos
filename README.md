# Rosa Analitica Website

Static marketing website for Rosa Analitica.

## Current version

The current site entry point is `index.html`, based on the latest available snapshot (`site_v2`).

## Stack

- HTML5
- Tailwind CSS via CDN
- Font Awesome via CDN
- Google Fonts via CDN

## Project structure

```text
.
|-- assets/
|-- index.html
|-- README.md
|-- VERSION_HISTORY.md
`-- archive/
    |-- site_v0.html
    |-- site_v1.html
    `-- site_v2.html
```

## Version archive

Older pre-Git snapshots were preserved in `archive/` so the project keeps a source-of-truth history even though the repository was created later.

## Running locally

Because this is a static site, you can preview it by opening `index.html` in a browser.

If you want a local server instead of opening the file directly, one simple option is:

```powershell
cd C:\Users\vikso\Projetos
python -m http.server 8000
```

Then open `http://localhost:8000`.

## Notes

- The repository import included the HTML snapshots provided by the user.
- The image assets from the shared OneDrive folder were imported into `assets/` and the HTML files were repointed to those local paths.
- See `VERSION_HISTORY.md` for the reconstructed change log between `v0`, `v1`, and `v2`.