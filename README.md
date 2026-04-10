# tiyatro.cls

A XeLaTeX document class for typesetting Turkish stage play scripts.

Designed to keep the `.tex` source **readable by playwrights**, not just programmers — character dialogue looks like dialogue, not code.

> Built on top of [`theatre.cls`](https://github.com/DavideFauri/theatre) by David Fauri.

---

## Features

- **Persona system** — declare characters once, use short aliases everywhere
- **Dialogue formatting** — character name in margin, dialogue text aligned
- **Chained stage directions** — `\alido\aysedo\mustdo Girerler.` → *(Ali, Ayşe, Mustafa: Girerler.)*
- **Inline character references** — mention another character inside dialogue, name auto-updates if you rename them
- **Group characters** — narrators, choruses, ensembles
- **Two-pass cast list** — each scene automatically lists its characters (requires two compilations)
- **Draft / Final modes** — line numbers, 1.5× spacing, "TASLAK" watermark in draft; clean output in final
- **Turkish language support** — correct smallcaps İ/ı, font fallback chain
- **Multilingual inline text** — `\ru{...}` for Cyrillic, `\en{...}` for English passages

---

## Requirements

- **XeLaTeX** (does not work with pdfLaTeX)
- Fonts: Aptos or Calibri (draft), Times New Roman (final) — falls back to TeX Gyre if unavailable

---

## Quick Start

```latex
\documentclass[draft]{tiyatro}   % or [final]

\title{Oyun Adı}
\author{Yazar Adı}
\draft{0.1}

% Declare characters
\karakter[Maria][Leningrad Komsomol'ü Sekreteri]{MARİA}{ma}
\karakter[Mustafa][Eski Devrimci]{MUSTAFA}{mu}

\begin{document}
\maketitle
\kisiler      % Cast list (Dramatis Personae)

\perde[Birinci Perde]
\sahne[Açılış]

\action{Işıklar yavaşça açılır.}

\mu{Merhaba! Buralara hep gelir misiniz?}
\ma{Selam. \ic{gülümser} Nasılsın?}

\mudo\mado Girerler.

\end{document}
```

Compile twice:
```
xelatex main.tex
xelatex main.tex
```

---

## Commands

### Characters

```latex
\karakter[Tam Ad][Açıklama]{SCRİPT ADI}{alias}
\karakter*[...]                          % starred: appears on cover page
\karaktergrubu[Grup Adı]{ \gkarakter... } % group (chorus, ensemble)
```

Each character gets:
- `\alias` — starts a new dialogue line
- `\aliasdo` — stage direction for that character
- `\aliasfont` — the character's font, for custom use

### Dialogue

```latex
\mu{Merhaba!}                    % dialogue line
\mu{\ma nerede?}                 % inline character reference → "Maria nerede?"
\mu{Ama sen _orada_ duruyordun!} % _word_ for emphasis
\mu{\ic{güler} Hayır olmaz!}     % \ic{} for inline action
```

### Stage Directions

```latex
\action{Işıklar söner.}          % anonymous stage direction
\mustdo Sahneye girer.           % single character
\mudo\mado\mustdo Girerler.      % chained — multiple characters
```

### Structure

```latex
\perde[Perde Adı]    % Act
\sahne[Sahne Adı]    % Scene
\stagedir{...}       % Large centered stage description
\beat                % Short pause — (Sus.)
```

### Multilingual

```latex
\ru{Добрый день}     % Cyrillic
\en{Good morning}    % English passage
```

### Inline Name Override

```latex
\setinlinename{ma}{Maria Hanım}  % override how \ma appears inside dialogue
```

---

## Draft vs Final

| | Draft | Final |
|---|---|---|
| Font | Aptos / Calibri | Times New Roman |
| Line spacing | 1.5× | 1× |
| Line numbers | ✓ | — |
| Cover | "TASLAK" label | Clean |

```latex
\documentclass[draft]{tiyatro}
\documentclass[final]{tiyatro}
```

---

## File Structure

```
tiyatro.cls        — main class (loads core + theme)
tiyatro-core.sty   — persona system, dialogue engine, scene/act system
tiyatro-theme.sty  — fonts, page layout, cover page, line numbers
main.tex           — example file
```

---

## Roadmap

**Branch A — Writing experience**
- Booklet print mode (A4 landscape, 4 script pages per sheet)
- Header/footer with page number, act name, version
- Page break protection (keep character name + lines together)
- `\sahne[Location — Time]{n}` optional argument
- `\beat` / `\sus` short pause marker
- Auto *(devam)* after stage directions

**Branch B — Copy modes**
- Personal copy (`\personalcopy{alias}`) — separate PDF per actor
- Rehearsal mode — starred character's lines left blank for memorization

---

## Contributing

Issues and pull requests welcome. See `CLAUDE.md` for architecture notes and implementation history.
