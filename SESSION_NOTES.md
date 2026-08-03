# Session notes: `grpo-slides` — "From MDPs to GRPO"

A LaTeX/Beamer slide deck teaching DeepSeek-R1's RL pipeline (Chapter 7, "Build a
DeepSeek Model from Scratch" — `deepseek-rl.pdf`, not committed to this repo), built
and iterated over one long session with Claude. This file exists so a future session
can pick up the thread without re-deriving all the context below.

## Files

- `grpo-slides.tex` — the deck source. Single file, ~800 lines, self-contained.
- `grpo-slides.pdf` — compiled output, 36 slides, 16:9.
- `cover-meme.png` — cold-open slide 1 (a meme), pulled in via `\includegraphics`.
  Slide 2 is the actual title slide; all `\eyebrow{}` numbers below assume the meme
  occupies slide 1.
- `deepseek-rl.pdf` — the source chapter (Manning book excerpt). **Not committed** —
  it's someone else's copyrighted book PDF; only reference it locally.

## How to build

```bash
tectonic grpo-slides.tex
```

`tectonic` (installed via Homebrew, `/opt/homebrew/bin/tectonic`) is a self-contained
LaTeX engine — no full TeX Live install needed, it fetches packages on first use and
caches them. No other build step. Re-run the same command after any edit; two passes
happen automatically when cross-references change.

To sanity-check after edits: `tectonic grpo-slides.tex 2>&1 | grep -E "warning|error"`.
A few sub-4pt `Overfull \vbox` warnings are long-standing and harmless (they predate
most of the content and don't visibly clip anything) — anything double-digit or an
`Overfull \hbox ... while \output is active` on *every* page is a real problem (see
Gotchas below).

## Design system (all defined at the top of the .tex)

- **Canvas**: 19.2 cm × 10.8 cm (1.5× beamer's tiny default 12.8×7.2cm 16:9 canvas —
  see Gotchas for how this is set correctly).
- **Palette**: `paper` #FAF9F4 (bg), `ink` #20242A (text), `inkdim` #5C6058 (muted),
  `accent` #8B6914 (ochre), `accentB` #1F6F68 (teal), `neg` #A24A3B (rust),
  `panel` #F1EFE6 (box fill), `linecol` #D8D5C9 (hairlines).
- **Fonts**: mathpazo (Palatino-alike, headlines + math), sourcesanspro (body),
  inconsolata (code/mono).
- **Frame tags** (`\frametag{color}{label}`, printed top-left under the eyebrow rule):
  `neg` = **Problem**, `accentB` = **Approach**, `accent` = **Result**,
  `inkdim` = **Building block**. This is the deck's core narrative device — see
  "Pedagogical structure" below.
- **Components**: `\eyebrow{section}{§ref}`, `\headline{}`, `\lede{}`, `\swatch{color}`
  (small color chip, used sparingly), `eqbox`/`eqboxhi` (equation boxes, `hi` =
  accent-bordered for emphasis), `callout`/`calloutwarm` (teal/ochre left-rule boxes,
  used for asides, caveats, "why this matters"), `card`/`cardA`/`cardB` (bordered
  info tiles, used in columns).
- **Diagrams**: two hand-built TikZ pieces reused across slides — `\trajdiagram`
  (state-chain with terminal reward, used on slides 2 and 8) and a branching-tree
  diagram (slide 9, LLM generation as a tree, wavy `decorations.pathmorphing` edges
  meaning "many single-token states compressed into one arrow"). Plus one-off TikZ
  diagrams: the classical MDP graph (slide 5), and the R1-family tree (slide 34).

## Pedagogical structure — the thing to preserve

The deck's spine is a repeated **Problem → Approach → Result** beat, explicitly
tagged on-slide, matching the book chapter's own rhetorical arc (confirmed by
re-reading the source): sparse reward → policy gradients → noisy reward → PPO →
value model expensive → GRPO → reward model needs labels → RLVR → R1-Zero → (poor
readability) → staged R1 pipeline. Every major restructuring in this session was in
service of making that chain *visible and unbroken* — see "Why things are the way
they are" below before changing the order of anything.

Within the PPO/GRPO section there's a second device: **building blocks**, tagged
`inkdim`, introduced once each and then referenced by name later (never re-derived):
J(θ) → Monte Carlo estimate of ∇J(θ) → one-token update → KL divergence → (bridge:
why a ratio rewrite) → what the clip does. The PPO and GRPO "assembled" slides each
show *one complete formula* with a `where` clause, plus a plain-text line naming
which earlier slide each symbol came from. Do not reintroduce the earlier
multi-color "swatch-coded formula" approach for these — it was tried and explicitly
rejected as too busy (see below).

## Full slide outline (36 slides)

1. Cover — meme (cold open, not part of the numbered content arc)
2. Title
3. The chain — the whole Problem→Approach→Result arc in one table (mirrors the
   overall structure below; update this if the chain changes)
4. Classical MDP — definitions (background, not in book)
5. Classical MDP — as a graph (TikZ; cleaning-robot example, V(s) on nodes)
6. Classical MDP — value functions, the Bellman-backup mental model (background)
7. LLM as MDP — same tuple, new content (§7.1)
8. **Problem** — reward only shows up at the end (§7.1.3)
9. LLM generation as a tree (TikZ; contrasts with slide 5's graph — no cycles,
   reward only at leaves; same 4 completions reused later in the GRPO example)
10. **Building block** — what is J(θ) (§7.2)
11. **Building block** — Monte Carlo estimate of ∇J(θ) (§7.2)
12. **Building block** — one sampled token, the compact update rule (§7.2.2)
13. **Problem** — raw reward is noisy → the advantage (§7.2.3)
14. **Building block** — KL divergence, what it means / how it's computed
15. Bridge — why PPO needs a rewrite (importance sampling, "not in the book")
16. Bridge — the ratio is block 3's gradient, in disguise (derivation)
17. **Problem** — the rewrite only holds up close to θ_old (defines "trust region")
18. **Building block** — what the clip actually does (min/clip, worked example)
19. Is this still mathematically correct? (honesty check: biased, pessimistic bound,
    TRPO connection — "not in the book")
20. PPO's baseline — "better than expected," not "good" (§7.3.2)
21. **Approach** — PPO, the complete update rule (§7.3.1)
22. **Problem** — the value model is a second network (§7.4)
23. **Approach** — GRPO intro (§7.4)
24. GRPO mechanism — group-relative advantage + Roger worked example (§7.4.1)
25. **Approach** — GRPO, the complete update rule (§7.4.2/7.4.3)
26. **Approach** — RLVR (§7.5)
27. One base model, two independent paths (DeepSeek-V3-Base, §7.6)
28. **Result** — R1-Zero, GRPO+RLVR applied directly (§7.6)
29. **Problem** — why "correct" isn't the same as "readable" (§7.6)
30. **Building block** — what is SFT (Supervised Fine-Tuning), contrasted with RL
31. **Approach** — DeepSeek-R1, built fresh, informed by R1-Zero's outputs (§7.7)
32. Which stage fixes which problem (table, ties back to slide 29's two problems)
33. **Approach** — DeepSeek-R1-Distill, smaller models, no RL at all (§7.7)
34. The whole R1 family, in one picture (TikZ family tree)
35. Implementation — one scalar loss, one backward pass (§7.8–7.10)
36. Summary — the whole lineage, one line each (§7.15)

## Why things are the way they are (chronological, condensed)

1. **Started as an HTML/JS artifact**, then the user explicitly asked for a real
   presentable PDF with LaTeX-typeset math — hence the switch to Beamer + `tectonic`.
2. **First full draft** followed the book fairly literally, including a full
   derivation of ∇J(θ) (log-derivative trick), Monte Carlo unbiasedness, etc. The
   user said this went deeper than the book and asked for simplification *and* for
   an explicit Problem→Approach mental model, since the book itself has that
   structure and the deck didn't surface it. That produced the tagged
   Problem/Approach/Result device and cut the heaviest derivation slides down to one
   light "policy gradient" slide.
3. User then asked specifically about **where KL divergence enters** the PPO/GRPO
   math and wanted the full objective written out, not just referenced — this
   restored some formality (full PPO/GRPO formulas with explicit KL terms) but kept
   it scoped to exactly what was asked.
4. A tangent into **what KL divergence conceptually means** (comparing two
   conditional distributions evaluated at the same state, not "the policy" as one
   fixed object) led to a request to rebuild the "building blocks → final formula"
   section from scratch, cleanly, in-deck. That's the origin of blocks 1–5, the two
   bridge slides, and the trust-region problem slide.
5. **Layout complaint**: cards and slides "look too short," cramped. Root cause
   found: default Beamer 16:9 canvas is 12.8×7.2cm, tiny — everything was being
   forced into scriptsize/footnotesize just to fit. Fixed by enlarging the physical
   canvas (see Gotchas — this took two attempts to get right).
6. User pushed back hard, correctly, on **whether clipping still gives a correct
   gradient** — this is a real, substantive point (clipping *does* zero the gradient
   in the capped region; PPO trades exactness for the TRPO-style pessimistic-bound
   stability guarantee). That became its own honesty-check slide (18) rather than
   being glossed over.
7. **"The min/clip formula doesn't make sense"** → added a dedicated worked-example
   table (A=+1, ratio=1.5 vs 0.7) showing the asymmetry directly: clipping caps
   credit for overshooting a good move, never caps the penalty for a bad one.
8. **Tried color-coding the assembled PPO/GRPO formulas** (teal/ochre/rust inline,
   with a swatch legend) to show which building block contributed which piece of
   the final formula. User feedback: still not simple enough, and specifically
   flagged that the clip was used on a slide before it was ever explained.
   **Response**: dropped the rainbow-formula approach entirely (it was the deck's
   only place using it, and it wasn't landing), replaced with plain single-color
   formulas plus a one-sentence plain-text attribution line ("ratio and clip are
   block 5; V_φ(s_t) is the value-model baseline from the last slide; …"), *and*
   fixed the actual ordering bug (moved the trust-region problem slide and the
   "what the clip does" slide before the assembled-formula slides; moved PPO's
   value-model-intuition slide before PPO's assembled slide, matching how GRPO's
   mechanism slide already correctly preceded GRPO's assembled slide).
9. User asked whether the "PPO controls drift with a ratio" story in the book text
   was in tension with the importance-sampling explanation given in chat. It
   isn't — the book states the *conclusion* (ratio + clip, motivated by
   drift/stability) without the *derivation*; the unclipped ratio objective is
   literally named `L^CPI` ("Conservative Policy Iteration") in the original PPO
   paper, citing Kakade & Langford's importance-sampling-based policy-improvement
   bound. This distinction — **book states the mechanism; several slides in this
   deck derive the *why* that the book skips** — is called out explicitly with
   "Not in the book" callouts on slides 14, 15(ish), and 18. Preserve that framing;
   don't blur book-sourced content with supplementary derivation.
10. Final stretch: user was confused about **whether R1-Zero, R1, and R1-Distill are
    the same model continued or different models**. Answer: three separate
    checkpoints, connected by a *data* flow (R1-Zero's filtered outputs feed R1's
    cold-start SFT data; R1's outputs feed R1-Distill's SFT data) not a *parameter*
    flow. This produced slides 26–33 (common starting point, dedicated R1 slide with
    an explicit "not the same weights" callout, dedicated R1-Distill slide, and the
    closing family-tree diagram with solid arrows = real training vs. dashed arrow =
    data-only influence). Also added the SFT building-block slide (§7.7 area) since
    "SFT" had been used as an unexplained acronym for several slides before this
    point.
11. Added a meme (`cover-meme.png`) as slide 1, purely as a cold open — every other
    slide shifted by one. If inserting/removing slides in the future, remember every
    `\eyebrow{...}` call hardcodes its own slide number (no auto-numbering macro
    exists for this); a full renumber is a one-line regex over
    `\eyebrow{...\textperiodcentered\ NN}` (see git history for the exact command
    used each time this happened).

## Gotchas (hard-won, don't redo the debugging)

- **Custom Beamer paper size**: use
  ```
  \usepackage{geometry}
  \geometry{paperwidth=19.2cm,paperheight=10.8cm}
  ```
  placed *after* `\documentclass[aspectratio=169]{beamer}`. Do **not** use
  `\setlength{\paperwidth}{...}` directly — Beamer keeps its own internal
  `\beamer@paperwidth` copy that a raw `\setlength` doesn't update, which produces a
  silent, identical-on-every-page `Overfull \hbox ... while \output is active`
  (background canvas / footline machinery misaligned with the real page size). The
  `geometry` package's `\geometry{}` command is the one that actually propagates
  correctly. Also don't pass `paperwidth=`/`paperheight=` as bare `\documentclass`
  options — they're silently ignored when `aspectratio` is also given; you get
  Beamer's plain default 4:3 size instead.
- **Don't load `enumitem` in a Beamer doc.** Beamer bundles its own legacy
  `enumerate.sty`; loading `enumitem` alongside it causes infinite macro recursion
  in `\labelenumi` (`TeX capacity exceeded, sorry [input stack size=5000]`). Use
  Beamer's native `enumerate`/`itemize` (bracket options like `[label=...]` still
  work via Beamer's own machinery) — this deck doesn't need enumitem for anything
  it currently does.
- **tcolorbox card heights**: fixed-height cards with two-line titles at
  `scriptsize`/`footnotesize` body text can overflow their own border silently
  (text spills below the box outline without a compile warning). If you shrink a
  `card`/`cardA`/`cardB` height, always visually check the rendered page — the
  compiler won't tell you.
- **PDF review workflow**: after `tectonic grpo-slides.tex`, use the `Read` tool
  directly on `grpo-slides.pdf` with a `pages` range (e.g. `pages: "16-17"`) to
  visually inspect specific slides — this is how essentially every layout bug in
  this session was actually caught (compiler warnings tell you *that* something
  overflows, not whether it looks acceptable).
- **Multi-page `pages` ranges** like `"18,23,28"` (comma list) sometimes only
  return the first page in practice — prefer single pages or contiguous ranges
  (`"18-20"`) if a request seems to silently return fewer pages than expected.

## Possible next steps (not started / explicitly deferred)

- Nothing is currently mid-edit; the deck compiles clean (only long-standing
  sub-4pt overfull warnings remain, all visually harmless).
- If asked to extend further: the RLVR section (slide 26) and Implementation
  section (slide 35) haven't been through the same "building blocks first" scrutiny
  as the PPO/GRPO section — worth checking if similar forward-reference issues
  exist there if the user asks for more depth in that area.
- No presentation dry-run / timing check has been done — deck has not been
  validated for live-talk pacing (36 slides).
