# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project overview

Org-roam knowledge base and blog exported to a Hugo static site (marek.onl).
This repo holds only the org sources and media. The Hugo site lives in
`~/marek.onl/`; export and deployment are driven by `~/repos/puke/puke.el`, an
Emacs package. Hosted on GitHub (`github.com:upbqdn/roam`).

`test-node.org` is a rendering testbed — ignore its contents as reference for
conventions, and exclude it from the label-ID invariant below.

## Publishing

There is no build/lint/test tooling in this repo. Publishing runs in Emacs
(via emacsclient):

- `puke-publish-note` — export the current note via ox-hugo, then deploy
- `puke-rebuild-notes` — re-export every note, then deploy
- Deploy pipeline: rsync `data/` into the site's `static/data`, Tailwind CSS
  build, `hugo`, pagefind index, rsync `public/` to the server
- `puke-pagefind-version` pins the pagefind release. Its docstring states the
  conditions under which a bump is safe; a bump made outside them breaks
  search for every returning visitor. Do not change it from memory.

## File conventions

- **Filenames:** kebab-case `.org` files
- **Front matter:** every note carries `:PROPERTIES:` with `:ID:`, `#+title:`,
    `#+date:`, `#+hugo_lastmod:`, `#+bibliography: ref.bib`,
    `#+cite_export: csl`, and `#+hugo_tags:` (one of `math`, `blog`, `crypto`).
    Math definitions use short keyword `:ID:`s (e.g. `ring`, `group`); blog
    posts use UUIDs. The only exception is `README.org` (the /about page),
    which has just `#+title:` and `#+export_file_name:`.
- **Update `#+hugo_lastmod:`** whenever you change a file's content.
- **Bibliography:** All citations go in `ref.bib` (BibTeX), referenced via
    `[cite:@key]`, exported with `#+cite_export: csl` and
    `#+print_bibliography:` at the end of the file. Files that cite nothing
    carry no `#+print_bibliography:`.

## Label IDs

Items such as definitions, theorems, proofs, examples, remarks, propositions,
algorithms, equations, figures, tables, listings, and etumons each get a
z-base-32 identifier. **Critical invariant:** label IDs must be unique and contiguous
across the entire repo — they are used for cross-file referencing.

Allocation is a single repo-wide counter in `id-counter`, which stores the
*last allocated* index. To allocate: increment the number in `id-counter`,
then encode the new value as z-base-32 (alphabet
`ybndrfg8ejkmcpqxot1uwisza345h769`, most significant digit first; e.g. 1 →
`b`, 32 → `by`). Index 0 (`y`) is reserved and must not be used. When adding
several items, allocate consecutive indices and leave `id-counter` at the last
one used.

Emacs helpers in puke: `puke-insert-anchor` (prompts for block type, allocates
an ID, inserts the full scaffold), `puke-insert-id`, `puke-release-id`
(reclaims the most recent unused ID), and `puke-insert-ref` (search anchors
repo-wide and insert a cross-reference).

## Blog posts vs definition files

**Blog posts** (`#+hugo_tags: blog`): narrative content with media, `#+date:`,
  `#+hugo_lastmod:`, and, when the post cites anything, `#+print_bibliography:` at the end. Body text should
  appear before the first heading — don't wrap the introduction in a heading.

**Math definitions** (e.g. `ring.org`, `group.org`): short files using custom
  blocks for structured content, tagged `math` (or `crypto` for the AES
  notes).

Custom block types and their label prefixes:
- `#+BEGIN_def` — definitions, anchored as `<<.def-X>>`
- `#+BEGIN_rem` — remarks, anchored as `<<.rem-X>>`
- `#+BEGIN_exp` — examples, anchored as `<<.exp-X>>`
- `#+BEGIN_prp` — propositions, anchored as `<<.prp-X>>`
- `#+BEGIN_thm` — theorems, anchored as `<<.thm-X>>`
- `#+BEGIN_alg` — algorithms, anchored as `<<.alg-X>>`
- `#+BEGIN_eqn` — equations, anchored as `<<.eqn-X>>`
- `#+BEGIN_fig` — figures, anchored as `<<.fig-X>>`
- `#+BEGIN_tab` — tables, anchored as `<<.tab-X>>`
- `#+BEGIN_lst` — listings, anchored as `<<.lst-X>>`
- `#+BEGIN_etu` — etumons, anchored as `<<.etu-X>>`
- `#+BEGIN_prf` — proofs, anchored as `<<.prf-X>>`

Where `X` is a z-base-32 identifier.

**Self-referencing pattern:** Each anchor is followed by a bold self-link, e.g.:
```
<<.def-g>> *[[.def-g][Definition g]]*. A *ring* is ...
```

**Tables** use the same self-referencing pattern with `#+begin_center`
nested inside the custom block:
```
#+BEGIN_tab
#+begin_center
<<.tab-9>> *[[.tab-9][Table 9]]*. Caption text.
#+end_center
| ... |
#+END_tab
```

**Figures** put the anchor above the image and the self-link caption below it:
```
#+BEGIN_fig
#+begin_center
<<.fig-x>>
[[file:/data/topic/name.svg]]
*[[.fig-x][Figure x]]*. Caption text.
#+end_center
#+END_fig
```

**Equations** carry no bold self-link; the anchor sits alone above display
math whose `\tag` repeats the ID:
```
#+BEGIN_eqn
<<.eqn-ro>>
\[ ... \tag{ro}\]
#+END_eqn
```

The site's MathJax config defines no custom TeX macros (only AMS tags), so
only standard LaTeX commands may be used in math.
Do not use `#+caption:` — it triggers Hugo's auto-numbering which conflicts
with the z-base-32 IDs.

**Cross-referencing:** `[[.def-g][Definition g]]` for anchors within the same
file, `[[file:group.org::.def-r][monoid]]` for cross-file references.

**Math notation:** prefer `\(...\)` and `\[...\]` over `$...$` and `$$...$$`.

## Media

- Media lives in `data/` with subdirectories per topic (e.g. `data/mining/`,
  `data/evolutionary-algorithms/`)
- Images: `[[file:/data/mining/name.jpg]]`
- Videos: `#+attr_html: :controls t` followed by a `#+begin_video` block:
  ```
  #+attr_html: :controls t
  #+begin_video
  <source src="/data/mining/name.mp4" type="video/mp4">
  #+end_video
  ```
- `data/**/*.mp4` is gitignored; commit images, SVGs, and GIFs but not videos
