---
date: '2026-07-14T07:00:00+05:30'
draft: true
title: 'Kitchen Sink: Feature Test Page'
description: "A throwaway page exercising every markdown, math, image, and diagram feature the site supports, so breakage shows up here instead of in a real post."
---

This page is scaffolding, not writing — it exists to break in exactly the ways a real post would, before a real post does. Every section below maps to one feature the site is expected to support. Delete this post before going live (it stays `draft: true` for exactly that reason).

## Text formatting

Plain paragraph text, with **bold**, *italic*, ***bold italic***, ~~strikethrough~~, and `inline code`. A link to [an external site](https://gohugo.io/) and a footnote reference[^1].

> A blockquote, for asides and caveats.
>
> > A nested blockquote, for asides on asides.

## Lists

Unordered, nested:

- First-level item
- Another item
  - Nested item
  - Another nested item
- Back to first level

Ordered:

1. Set initial conditions
2. Step the integrator
3. Check termination condition

Task list:

- [x] Scaffold the site
- [x] Fix theme/Hugo version mismatch
- [ ] Write the actual RK4 post

## Horizontal rule

---

## Table

| Method       | Order | Error per step | Cost per step |
|--------------|:-----:|:---------------:|:--------------:|
| Euler        | 1     | O(h²)           | 1 eval          |
| Midpoint     | 2     | O(h³)           | 2 evals         |
| RK4          | 4     | O(h⁵)           | 4 evals         |

## Code blocks

Python, for the simulator itself:

```python
def rk4_step(f, x, t, h):
    k1 = f(x, t)
    k2 = f(x + 0.5 * h * k1, t + 0.5 * h)
    k3 = f(x + 0.5 * h * k2, t + 0.5 * h)
    k4 = f(x + h * k3, t + h)
    return x + (h / 6.0) * (k1 + 2 * k2 + 2 * k3 + k4)
```

Bash, for tooling:

```bash
hugo new content posts/rk4-ascent-simulator/index.md
hugo server -D
```

YAML, for config:

```yaml
params:
  math: true
```

## Math

Inline math sits in a sentence like this: the state vector $\mathbf{x} = [h, v, m]^T$ evolves under $\dot{\mathbf{x}} = f(\mathbf{x}, t)$.

Display math, the RK4 update:

$$\mathbf{x}_{n+1} = \mathbf{x}_n + \frac{h}{6}\left(k_1 + 2k_2 + 2k_3 + k_4\right)$$

A matrix, since ascent dynamics end up linearized somewhere eventually:

$$
A = \begin{bmatrix} 0 & 1 \\ -\omega^2 & -2\zeta\omega \end{bmatrix}
$$

A sum and an integral, for good measure:

$$\sum_{i=1}^{n} \frac{1}{i^2} \quad \text{and} \quad \int_0^T v(t)\,dt$$

A literal dollar amount, escaped so it doesn't get treated as a math delimiter: this whole blog runs on a budget of \$0.

## SVG: data plot

Generated with matplotlib and saved as `.svg` — vector, not a screenshot.

{{< figure src="altitude-profile.svg" caption="Altitude vs. time, 100 s burn, h = 0.01 s (toy data, not real physics)" >}}

## SVG: diagram

Same tool (matplotlib), used for a box-and-arrow diagram instead of a data series — proves SVG embedding doesn't care whether the source was a plot or a diagram.

{{< figure src="pipeline-diagram.svg" caption="Signal flow: state feeds the integrator, which queries dynamics each substep" >}}

## Raster image via ImageMagick

A synthetic "screenshot" (`console-screenshot.png`, 34.2 KB) run through `magick console-screenshot.png -quality 82 console-screenshot.webp`, producing a 6.4 KB WebP — about 5.2× smaller. Only the `.webp` ships; the source PNG was deleted after conversion.

{{< figure src="console-screenshot.webp" caption="Mock simulator console, PNG&#8594;WebP at quality 82" >}}

## Internal link (ref shortcode)

As derived in [the RK4 post]({{< ref "posts/rk4-ascent-simulator" >}}), the update rule is a weighted average of four slope estimates. If that target didn't exist, this build would have failed — that's the point of using `ref` instead of a hand-written URL.

## Footnotes

[^1]: This is the footnote body. Hugo/Goldmark renders these as a numbered list at the bottom of the page with back-links.
