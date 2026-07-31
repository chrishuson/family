# Huson Family Website — Project Notes

**Owner:** Christopher Huson
**Repo:** github.com/chrishuson/family — **public** (see [Privacy](#privacy) below)
**Live site:** <https://family.huson.com> (GitHub Pages, served from `/docs` on `main`)

Shared conventions live in `~/Documents/Tools/Tech_Stack_And_Conventions.md`;
the cross-project map is `~/Documents/Tools/project_registry.md`.

---

## Purpose

A small static site for the extended Huson family: trip pages (ski trips,
Christmas travel), grocery and packing lists, and typeset genealogy charts.
Pages are hand-written Markdown; GitHub Pages renders them with Jekyll.

---

## Layout

| Path | Contents |
| --- | --- |
| `docs/` | Everything published — Jekyll's source root |
| `docs/index.md` | Site home page (currently the Park City 2026 ski trip) |
| `docs/xmas2026.md`, `docs/xmas2026-flights-dr.md` | Christmas 2026 Dominican Republic trip |
| `docs/ski-trip-history.md`, `docs/groceries.md` | Recurring reference pages |
| `docs/images/` | Photos used by the trip pages |
| `docs/*.tex` / `docs/*.pdf` | Genealogy charts (see below); PDFs are committed and linkable |
| `docs/_config.yml` | Jekyll config — `relative_links` enabled |
| `docs/CNAME` | `family.huson.com` |

There is no build step and no Gemfile. Push to `main` and GitHub Pages rebuilds.

---

## Page conventions

- **Internal links omit the `.md` extension** — `[ski trip history](ski-trip-history)`.
  This works because `relative_links` is enabled in `_config.yml`. Do not write
  `ski-trip-history.md` or a leading `/`.
- **Images:** Markdown `![alt](file.jpg)` for simple cases; raw
  `<img src="images/x.jpg" alt="..." width="85%">` when the image needs sizing.
  Trip photos go in `docs/images/`; older loose images sit at the `docs/` root.
- **Markdown style:** follow the global rules in `~/.claude/CLAUDE.md` — escape
  every `$` as `\$`, blank lines around headings and lists, spaces around pipes
  on table separator rows (`| --- | --- |`).
- Commit messages in this repo are short and lowercase ("add DR flights page").

---

## Genealogy charts (LaTeX)

Built with the `genealogytree` package (`template=signpost`), compiled by
LaTeX Workshop in VS Code or `pdflatex` on the command line. The committed PDFs
are what family members actually open, so **rebuild and commit the PDF whenever
the `.tex` changes**.

| Source | Chart |
| --- | --- |
| `john+maria_descendants.tex` | John & Maria's descendants, full legal names |
| `john+maria_descendants-simple.tex` | Same tree with first names / nicknames |
| `john-ancestors.tex`, `maria-ancestors.tex` | Ancestor charts (converted to the paper-size pattern below) |
| `andrew-ancestors.tex` | **Experimental draft — the data in it is not correct.** Chris started it to play with the layout and never finished. Do not convert it to the paper-size pattern, do not "fix" its formatting, and do not treat it as a source of family facts. |
| `john-contemporaries.tex`, `john-contemp_mixed-trees.tex` | Sibling / cohort charts |
| `descendants-package-example.tex` | Scratch file for trying `genealogytree` features |

### Paper size is a build-time switch

Four charts have been converted to this pattern: `john+maria_descendants`,
`john+maria_descendants-simple`, `maria-ancestors`, and `john-ancestors`. Each
captures its `tikzpicture` into a save box and prints it through `\ShowTree`,
which scales the tree to fill the text block — limited by width or by height,
whichever binds first:

```latex
\newcommand{\ShowTree}{%
  \sbox{\FitTree}{\resizebox{\textwidth}{!}{\usebox{\RawTree}}}%
  \ifdim\dimexpr\ht\FitTree+\dp\FitTree\relax>\TreeHeight
    \sbox{\FitTree}{\resizebox{!}{\TreeHeight}{\usebox{\RawTree}}}%
  \fi
  \usebox{\FitTree}%
}
```

The height check matters: the descendants tree is a 3.5:1 ribbon and is always
width-bound, but the ancestor charts are nearly square (1.7:1) and overflow onto
a second page if only the width is constrained. `\TreeHeight` is
`\textheight - 6\TitleSize`, reserving room for the title block. Do not use
`adjustbox` with `min size` + `max size` to do this — that combination hangs
`pdflatex`.

Paper size comes from a single `\Paper` macro with a `\providecommand` default
(legal landscape), overridable from the command line:

```bash
cd docs

# Default: US legal landscape, 14 x 8.5 in
pdflatex -synctex=1 john+maria_descendants.tex

# Tabloid / ledger, 17 x 11 in
pdflatex -jobname john+maria_descendants-17x11 \
  '\def\Paper{paperwidth=17in,paperheight=11in,margin=0.5in}\input{john+maria_descendants}'

# Wall poster on a 36 in roll
pdflatex -jobname john+maria_descendants-36x14 \
  '\def\Paper{paperwidth=36in,paperheight=14in,margin=1in}\input{john+maria_descendants}'
```

Never hand-tune the `firstlevel` / `secondlevel` / `thirdlevelwide` box widths to
make a chart fit a page — change the paper and let `\ShowTree` do it. Those
widths exist only to control *relative* box proportions within the tree. After
changing paper, **check the page count**: a chart that silently runs to two pages
is the failure mode this pattern is guarding against.

`\TitleSize` is `\textwidth/60` in every converted chart, so charts printed on the
same paper get identical title sizes and hang as a matched set.

### Centering the top couple under the title

`genealogytree` positions the root couple over the pivot of the family below it,
not over the tree's bounding box — so on the descendants charts the couple sits
noticeably left of the centered title. Fix it with a family-level `pivot shift`
on the root `child{...}`, as in `john+maria_descendants-simple.tex`:

```latex
child[pivot shift=-2.76cm]{
  g[id=john, firstlevel]{Grandpa John}
  p[id=maria, firstlevel]{Gran Maria}
```

Negative shifts move the couple **right**; positive move it left. This moves only
the couple — the tree's bounding box is unchanged — and the value is in tree
coordinates, so it survives any `\Paper` size and any `\ShowTree` scale. Re-tune
it only if the tree's shape changes (a branch added or removed). To measure:
`pdftotext -bbox <pdf> -` gives word positions; centre the couple's box pair on
half the page width.

### Printing the family trees

Natural (unscaled) sizes, which is what drives everything below:

| Chart | Natural ink | Aspect | Name text |
| --- | --- | --- | --- |
| Descendants (full legal names) | 14.7 x 4.2 in | 3.5:1 ribbon | 8 pt throughout |
| Descendants (simple / nicknames) | 14.0 x 4.2 in | 3.4:1 ribbon | 8 pt throughout |
| Maria ancestors | 13.3 x 7.7 in | 1.7:1 | 8 / 7 / 6 pt (auto-shrunk by `genealogytree`) |
| John ancestors | 13.8 x 7.6 in | 1.8:1 | 8 / 7 / 6 pt |

The descendants ribbon is why the chart overflowed 8.5 x 14 legal paper before
the fix. Committed sizes:

| PDF | Paper | Scale | Name text | Use |
| --- | --- | --- | --- | --- |
| `john+maria_descendants-simple-17x11.pdf` | 17 x 11 in tabloid | 1.15x | ~9.2 pt | **The hanging set** |
| `maria-ancestors-17x11.pdf` | 17 x 11 in tabloid | 1.21x | ~9.7 pt | **The hanging set** |
| `john-ancestors-17x11.pdf` | 17 x 11 in tabloid | 1.17x | ~9.3 pt | **The hanging set** |
| `john+maria_descendants-17x11.pdf` | 17 x 11 in tabloid | 1.09x | ~8.7 pt | Full-legal-names alternative to the simple chart |
| `john+maria_descendants-36x14.pdf` | 36 x 14 in | 2.08x | ~17 pt | Best value wall chart — fills a 36 in roll with no wasted paper |
| `john+maria_descendants-36x24.pdf` | 36 x 24 in | 2.32x | ~18 pt | Standard orderable poster size; same tree, more white space |
| `john+maria_descendants.pdf`, `-simple.pdf` | 14 x 8.5 in legal, landscape | ~0.93x | ~7.4-7.8 pt | Home printer / legal tray |
| `maria-ancestors.pdf`, `john-ancestors.pdf` | 14 x 8.5 in legal, landscape | ~0.97x | ~7.8 pt | Home printer / legal tray |

**The three-chart hanging set** is the *simple* (nickname) descendants chart plus
both ancestor charts, all at 17 x 11: 9.2 / 9.7 / 9.3 pt — within 5% of each
other, effectively identical on a wall. All three fill the same 16.07 in of sheet
width with identical margins and identical title size, so the sheets line up. The
descendants sheet uses only 5.8 in of its 11 in height (the tree is a ribbon), so
it floats in the middle of its frame while the ancestor sheets nearly fill theirs.
Mat the descendants print or trim it if the imbalance bothers you.

**Where to print (Upper East Side):**

- **FedEx Office Print & Ship Center**, 1337 Lexington Ave (at 89th St),
  (212) 348-5478 — self-serve 11x17 plus full-service large format. Wide-format
  black-and-white bond ("engineering print") is the cheap option; heavyweight
  matte poster stock costs more. Standard poster sizes are 16x20, 18x24, 22x28,
  24x36, 36x48; custom lengths on a 36 in roll are normal for bond prints.
- **Staples**, 1280 Lexington Ave (at 86th St), (212) 426-6190 — self-serve
  copiers do 11x17; the Print & Marketing desk does engineering prints up to
  36 in wide.

Bring a **USB stick with the PDF** or email it ahead — do not let the counter
scale it, since the PDFs are already sized exactly to the sheet. Ask for
**no auto-fit / print at 100%**.

---

## Privacy

`docs/_config.yml` describes the site as "For use by Huson family members and
friends only," but the repository and the Pages site are **public**. Anyone can
read the full legal names and birth-order of the grandchildren and
great-grandchildren in the genealogy PDFs, and the flight numbers, dates, and
lodging addresses on the trip pages.

Assume anything committed here is world-readable. Before adding new material,
check it against that assumption — especially children's full names, addresses,
confirmation numbers, and travel dates. Making the repo private would also take
the site down unless the account has a GitHub plan that allows Pages on private
repos.

---

## Working agreements

- Do not commit or push without being asked; Chris reviews and pushes.
- Rebuild the chart PDF after editing a `.tex`, and delete the `.aux` / `.log`
  droppings. `.synctex.gz` files are tracked in this repo — regenerate them with
  `-synctex=1` rather than leaving them stale.
- Trip planning research and drafts live in `~/Documents/Travel/`; this repo
  holds only the finished pages that go on the website.
