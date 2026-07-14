---
date: '2026-07-13T23:14:44+05:30'
draft: true
title: 'Rk4 Ascent Simulator'
---

The state propagates as $\dot{\mathbf{x}} = f(\mathbf{x}, t)$, and RK4 gives

$$\mathbf{x}_{n+1} = \mathbf{x}_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)$$

As derived in [the RK4 post]({{</* ref "posts/rk4-ascent-simulator" */>}}), ...