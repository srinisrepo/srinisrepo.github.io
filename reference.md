# Blog Operations Manual

Reference for running the blog. Commit this file to the **repo root** (next to `hugo.yaml`).
Anything outside `content/` is not published, so this stays private to the repo.

---

## 0. Quick reference — the 90% case

```powershell
# 1. Scaffold a new post (creates a folder = page bundle)
hugo new content posts/my-post-slug/index.md

# 2. Write with live preview (auto-reloads on save)
hugo server -D
# → http://localhost:1313

# 3. When done: in the post's front matter, set
#    draft: false

# 4. Publish
git add .
git commit -m "Publish: my post slug"
git push
# Live at https://srinisrepo.github.io in ~60 seconds.
```

Slug rules: **lowercase, hyphens, descriptive, no dates.** The folder name IS the URL forever. `rk4-ascent-simulator` → `/posts/rk4-ascent-simulator/`.

---

## 1. Anatomy of a post

Every post is a **folder** (a "page bundle"), not a lone file. Its images live inside it.

```
content/posts/rk4-ascent-simulator/
├── index.md              ← the article (must be named index.md)
├── error-convergence.svg ← plots, referenced by relative path
├── trajectory.webp
└── freebody-sketch.svg
```

Delete the folder → the post and all its assets are gone together. Nothing orphaned.

### Front matter (the block between `---` at the top)

```yaml
---
title: "Building a 3DOF Ascent Simulator from an RK4 Integrator"
date: 2026-07-20T10:00:00+05:30   # set by hugo new; edit if needed
draft: true                        # THE publish switch. false = live on next push
summary: "One-sentence hook shown on the list page. Write it last."
tags: [simulation, control, numerical-methods]
math: true                         # ONLY needed if the post contains LaTeX
showtoc: true                      # table of contents; default is true site-wide
---
```

Notes:
- `draft: true` posts appear locally with `hugo server -D` but **can never leak to production** (`buildDrafts: false` in CI).
- `math: true` loads KaTeX for that page. Omit it on math-free posts — saves readers a script load.
- `tags` are the only taxonomy. Reuse existing tags; don't invent a new tag per post or the tag pages become useless.
- A post dated in the future won't publish until that date passes (`buildFuture: false`).

---

## 2. Linking

### To your own posts (ALWAYS this way)

```markdown
As derived in [the RK4 post]({{< ref "posts/rk4-ascent-simulator" >}}), the truncation error...
```

- The argument is the **folder path under `content/`**, no leading slash, no `.md` needed.
- If the target doesn't exist (typo, renamed post), **the build fails and tells you**. This is the feature. Never hand-write `/posts/...` URLs — those rot silently.
- Link to the about page: `{{< ref "about" >}}`.

### To a specific section (heading) of another post

Headings get automatic anchors (lowercased, hyphenated). To link to `## Step Size Selection` in another post:

```markdown
[choosing h]({{< ref "posts/rk4-ascent-simulator#step-size-selection" >}})
```

Same-page section link: `[see above](#step-size-selection)`.

### External links

Plain markdown: `[Hugo docs](https://gohugo.io/documentation/)`.
For references at the end of a post, footnotes read cleanly:

```markdown
The classic treatment is Hairer et al.[^1]

[^1]: Hairer, Nørsett, Wanner, *Solving Ordinary Differential Equations I*, Springer.
```

---

## 3. Images — every method, and when to use each

### Decision table: which format

| Content | Format | Why |
|---|---|---|
| Plots, graphs, charts | **SVG** | Vector: sharp at any zoom, tiny (20–100 KB of text), selectable text labels |
| Diagrams, sketches (drawn digitally) | **SVG** | Same reasons; export SVG from draw.io/Inkscape/Excalidraw |
| Screenshots, photos | **WebP** | 5–10× smaller than PNG at equal quality |
| Pixel-perfect UI captures with text | **PNG → WebP**, check readability; keep PNG only if WebP smears the text | rare |
| Animations | **Avoid GIF** (huge files). Use a video (see 3.6) or a sequence of stills | bandwidth |

### 3.1 Plots straight from code (the main workflow)

In matplotlib, save vector output directly into the post's folder:

```python
plt.savefig("content/posts/rk4-ascent-simulator/error-convergence.svg",
            bbox_inches="tight")
```

Then in `index.md`:

```markdown
![RK4 error convergence, slope 4 on log-log axes](error-convergence.svg)
```

The text in `[...]` is **alt text**. Always write it, describing what the figure shows —
it's what screen readers, broken loads, and search engines see.

### 3.2 Basic markdown image (relative path, same folder)

```markdown
![Free-body diagram of the vehicle in the pitch plane](freebody-sketch.svg)
```

Works for any file sitting in the post's bundle. This is the default method.

### 3.3 Figure with a caption (use for anything a reader might cite)

```markdown
{{< figure src="trajectory.webp"
    alt="Altitude vs downrange distance for the nominal ascent"
    caption="Nominal ascent trajectory. 100 s burn, h = 0.01 s, no wind."
    width="700" >}}
```

- `caption` renders under the image; `alt` is still required and is different (alt describes, caption comments).
- `width` (px) constrains oversized images; omit it normally.

### 3.4 Converting screenshots to WebP (do this BEFORE committing)

```powershell
magick screenshot.png -quality 82 screenshot.webp
```

Batch-convert everything in a folder:

```powershell
magick mogrify -format webp -quality 82 *.png
```

Rule of thumb: any raster image over ~300 KB in the repo is a mistake. Check with
`ls` before committing. Raw phone photos are 3–5 MB — never commit one unconverted.

### 3.5 Site-wide images (not tied to one post)

Files in the top-level `static/` folder are served from the site root:
`static/favicon.ico` → `https://srinisrepo.github.io/favicon.ico`.
Use for favicon and nothing else, basically. Post images belong in post bundles.

### 3.6 Video / animation (only if ever genuinely needed)

Raw HTML in markdown is stripped by default. To allow it, add once to `hugo.yaml`:

```yaml
markup:
  goldmark:
    renderer:
      unsafe: true    # allows raw HTML in markdown; fine for a single-author site
```

Then embed an mp4/webm placed in the post bundle:

```html
<video controls muted loop width="700">
  <source src="ascent-animation.webm" type="video/webm">
</video>
```

Keep videos short and small (a 10 s webm ≈ 1–2 MB). For YouTube instead:
`{{< youtube VIDEO_ID >}}` — costs the repo zero bytes.

---

## 4. Math

Requires `math: true` in the post's front matter.

- Inline: `$\dot{\mathbf{x}} = f(\mathbf{x}, t)$` → renders inside the sentence.
- Display (own line, centered):

```markdown
$$
\mathbf{x}_{n+1} = \mathbf{x}_n + \frac{h}{6}\left(k_1 + 2k_2 + 2k_3 + k_4\right)
$$
```

- Also valid: `\( ... \)` inline and `\[ ... \]` display.
- **Literal dollar sign in prose:** write `\$` ("costs \$5"), or a stray `$` will pair with
  the next one and typeset your paragraph as garbage math.
- Multi-line derivations: use `aligned` inside `$$`:

```markdown
$$
\begin{aligned}
k_1 &= f(t_n, x_n) \\
k_2 &= f(t_n + \tfrac{h}{2},\; x_n + \tfrac{h}{2}k_1)
\end{aligned}
$$
```

- If math shows as raw `$...$` text: (1) `math: true` present? (2) restarted `hugo server`
  after any config change? (3) delimiters balanced?

---

## 5. Code blocks

Always specify the language for syntax highlighting:

````markdown
```python
def rk4_step(f, t, x, h):
    k1 = f(t, x)
    ...
```
````

- Inline code: backticks — `` `hugo server -D` ``.
- Languages you'll use: `python`, `cpp`, `yaml`, `powershell`, `text` (for plain output).
- Readers get a copy button automatically (`ShowCodeCopyButtons: true`).
- Long listings: show the interesting 20 lines in the post, link the full file
  (GitHub repo link) — a post is not a code dump.

---

## 6. General writing structure (house style)

Per the Engineering Blog Principles — the mechanical version:

1. **Open with the problem**, not biography. First paragraph: what this post lets the reader do.
2. **`## `-level headings** as the skeleton (H1 is the title; never use `#` in the body).
   The TOC is generated from these — write headings that read as a summary by themselves.
3. State **confidence levels inline**: "current understanding", "unverified", "I suspect".
4. **Show failures** in their own section, not a footnote. What broke, why, what fixed it.
5. Every equation introduced → every symbol defined at first use.
6. Every figure → referenced from the prose ("Figure shows..."); no orphan images.
7. **End with open questions** — a final `## Open questions` section, always.
8. `summary:` front matter written last, after you know what the post actually became.

### Pre-publish litmus (from the principles — answer all five)

1. Did I learn something nontrivial creating this?
2. Still useful in five years?
3. Could someone reproduce my results from this alone?
4. Insight not on the first page of a search?
5. Proud if a professor/recruiter/collaborator reads this first?

### Technical pre-publish checklist

- [ ] `draft: false`
- [ ] Site builds clean locally: run `hugo` (no `-D`) once — zero errors, zero warnings
- [ ] All internal links use `ref` (build passing proves they resolve)
- [ ] All images: correct format per the table, raster ones < 300 KB, alt text on every one
- [ ] Math renders in local preview (check both inline and display)
- [ ] `summary:` and `tags:` filled
- [ ] Read the whole post once in the browser, not the editor

---

## 7. Updating an old post

Principles require revisiting old posts. Mechanically trivial:

1. Edit the `index.md`, push. Done — same pipeline.
2. For a substantive revision, add a visible note at the top of the content:
   > **Updated 2026-09-14:** corrected the step-size analysis; original conclusion overstated stability margin.
   Honest changelog > silent rewrite (credibility comes from accuracy, not certainty).
3. Do **not** change the slug/folder name — that breaks every inbound URL. The URL is a promise.

---

## 8. Troubleshooting

| Symptom | Cause → Fix |
|---|---|
| Pushed, site not updating | Check repo → **Actions** tab. Red? Open log, find the red step. Green but stale page? Wait 2 min, then hard-refresh `Ctrl+F5` |
| Build fails: `REF_NOT_FOUND` | A `ref` shortcode points to a slug that doesn't exist. Error message names the file and line. Fix the path |
| Post invisible on live site | `draft: true` still set, or `date:` in the future |
| Post invisible even locally | Not running with `-D`, or file isn't named `index.md`, or folder isn't under `content/posts/` |
| Math shows as raw `$..$` | `math: true` missing in front matter; or server not restarted after config edit |
| `hugo` not recognized | New terminal session needed after install; PATH issue |
| Theme broken / site unstyled after cloning repo on a new machine | Submodule not pulled: `git submodule update --init --recursive` |
| "pages build and deployment" run fails with Jekyll logs | Harmless legacy pipeline; ensure Settings → Pages → Source = **GitHub Actions**, then ignore |
| Actions warning about Node versions | Cosmetic, not yours. Ignore |

### Setting up on a new machine (the whole thing)

```powershell
winget install Git.Git
winget install Hugo.Hugo.Extended        # then check: hugo version → must say "extended"
winget install ImageMagick.ImageMagick
git clone https://github.com/srinisrepo/srinisrepo.github.io.git blog
cd blog
git submodule update --init --recursive   # ← the step everyone forgets
hugo server -D
```

---

## 9. Standing rules (violating these creates the maintenance you said you didn't want)

1. **Never edit anything in `themes/PaperMod/`.** Extensions go in your own `layouts/partials/` (that's how the KaTeX hook works).
2. **Never commit `public/`** — it's in `.gitignore`; keep it there.
3. **Never unpin versions** (PaperMod `v7.0` submodule, Hugo `0.145.0` in `.github/workflows/hugo.yml` and on your machine). If you ever deliberately upgrade, change both Hugo versions together, verify locally, then push.
4. **Images: convert before commit.** The repo history keeps every byte forever — a deleted 5 MB photo still bloats every future clone.
5. **URLs are permanent.** Never rename a published post's folder.
6. The blog is infrastructure. Infrastructure time is capped at: reading this file.
