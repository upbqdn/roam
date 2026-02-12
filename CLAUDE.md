# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Org-roam knowledge base and blog exported to a Hugo static site. Contains mathematical definitions (algebraic structures) and blog posts. Hosted on GitLab (`gitlab.com:upbqdn/roam`).

## File conventions

- **Filenames:** kebab-case `.org` files
- **IDs:** Every file has a `:PROPERTIES:` block with `:ID:` (UUID or short keyword like `ring`)
- **ID counter:** `id-counter` tracks the next available label index
- **Bibliography:** All citations go in `ref.bib` (BibTeX), referenced via `[cite:@key]`, exported with `#+cite_export: csl` and `#+print_bibliography:`

## Blog posts vs definition files

**Blog posts** (`#+hugo_tags: blog`): narrative content with media, `#+date:`, `#+hugo_lastmod:`, and `#+print_bibliography:` at the end.

**Math definitions** (e.g. `ring.org`, `group.org`): short files using custom blocks for structured content:
- `#+BEGIN_def` / `#+BEGIN_rem` / `#+BEGIN_pr` / `#+BEGIN_ex` with labeled anchors like `<<.df-m>>`
- Internal links: `[[.df-m][Definition m]]` (anchors), `[[id:ring][ring]]` (org-roam)
- Math notation: prefer `\(...\)` and `\[...\]` over `$...$` and `$$...$$`

## Media

- Media lives in `data/` with paths like `/data/mining/name.jpg`
- Images: `[[file:/data/mining/name.jpg]]`
- Videos: `#+begin_video` blocks with `<source src="/data/mining/name.mp4" type="video/mp4">` and `#+attr_html: :controls t`
- `data/*.mp4` is gitignored; only subdirectory media is tracked

## Remote execution

Per the user's global instructions, run commands on the remote host `d` via:
```
ssh d 'fish -c "cd /same/path/as/on/localhost; and <command>"'
```
All repos are rsynced to `d` under identical paths on each file save.
