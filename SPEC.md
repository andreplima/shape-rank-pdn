# Shape-Rank-PDN — Functional & Technical Specification

**Version:** 1.0 (deployed as `index.html` on GitHub Pages, November 2026)
**Scope:** This document describes what the Shape-Rank-PDN applet does, how it is structured, and the design choices behind it. It is meant for anyone who maintains, extends, or learns from the codebase — authors of lecture materials who want to adapt the applet, students taking it apart to understand it, reviewers assessing its claims, and the author's future self.

---

## 1. Overview

### 1.1 Purpose

Shape-Rank-PDN is a single-page interactive applet that lets a user reshape a **radial polygon** $z$ on the unit disc — one vertex per axis, as in a radar chart — and see, in real time, its decomposition into **eigenpolygons** $z = \sum_j E_j$, together with the coverage of each polygon on a **polar lattice** of equal-area cells. The applet is designed to make three facts visible:

1. every polygon is a sum of regular, possibly star or degenerate, polygons;
2. total signed area decomposes accordingly, $A(z) = \sum_j A(E_j)$;
3. the cell-by-cell coverage does **not** decompose, neither cell by cell nor aggregated by annulus or sector.

### 1.2 Target audience

- Undergraduate and early-graduate students learning the discrete Fourier transform, circulant matrices or the geometry of polygons.
- Instructors preparing lectures or classroom demonstrations.
- Researchers studying the relationship between diagram geometry and feature-matrix rank in learning tasks.

### 1.3 In scope

- Radial polygons with $d \in \{3, \ldots, 10\}$ vertices, one per axis $\zeta_k = e^{2\pi i k/d}$, with radii quantised to $B^* = \{1/g, \ldots, 1\}$, $g \in \{2, \ldots, 10\}$.
- Polar lattices with $n_{spd} \in \{1, \ldots, 4\}$ sectors per natural wedge and $n_a \in \{1, \ldots, 8\}$ equal-area annuli.
- Exact computation of the discrete Fourier coefficients, eigenpolygons and their signed areas.
- Exact computation of three cell measures (signed, unsigned, total-variation) for $z$ and every eigenpolygon, including self-intersecting and clockwise ones.
- Numerical rank of every coverage matrix.
- Four guided exercises, a numeric-details inspector, and export of all tables to an `.ods` spreadsheet.

### 1.4 Out of scope

- Polygons whose vertices are not on the $d$ axes (general $d$-gons).
- Lattices other than the equal-area polar lattice.
- Session persistence (reload resets to defaults).
- URL-encoded state sharing.
- Mobile/touch optimisation (functional but not tuned).
- Programmatic APIs.

---

## 2. Functional specification

The applet is divided into two side-by-side panels. The left panel (30% of the width) holds the shape and lattice controls; the right panel (70%) holds the decomposition. Both panels update whenever any input changes, including continuously while a vertex is being dragged.

### 2.1 Left panel — Shape & lattice

**Header.** The panel title "Shape & lattice" sits at the top-left; an "Instructions ↗" link on the top-right opens the project README (GitHub repo) in a new tab.

**Drawing area.** A square SVG shows:

- The unit circle.
- The polar lattice in thin light-gray lines: $n_a - 1$ inner circles of radius $\sqrt{m/n_a}$, and $S = d \cdot n_{spd}$ rays. Every $n_{spd}$-th ray is a natural axis $\zeta_k$ and is drawn slightly darker.
- Small gray ticks on each axis at the positions of $B^*$.
- The polygon $z$, filled in Dutch orange (`#F36C21`, 45% opacity) with a magenta edge.
- One blue marker per vertex, and a label $z_k$ (with subscript) just outside the unit circle on each axis.

**Vertex manipulation.** Each vertex is draggable along its own axis only: the pointer is projected onto the ray $\zeta_k$ and the radius clamped to $[1/g, 1]$. During a drag the polygon and every derived view update continuously. On release the radius snaps to the nearest value in $B^*$, so $a_k$ is never 0. Vertices keep their index; they are never re-sorted. A transient status message confirms the snapped value (e.g. `a1 = 1/2`).

**Sliders.** Four range inputs: $d$ (3–10, default 4), nspd (1–4, default 2), na (1–8, default 4) and granularity $g$ (2–10, default 4). Changing $d$ or $g$ resets $a$ to all ones, i.e. to the regular $d$-gon, and says so in the status line; changing nspd or na leaves $a$ unchanged.

**Reset button.** Restores the four slider defaults and $a$ = all ones. It does not change the checkbox, the drop-down or the scale of the coefficient chart.

**Status message.** A single-line area next to the Reset button shows transient feedback (green for confirmations, red for errors such as a failed export), auto-clearing after ~2.5 seconds.

**Hover.** Hovering over a lattice cell shows a tooltip with its index (sector $s$, annulus $m$), its measure for $z$ under the selected sense, and that measure divided by the cell area.

### 2.2 Right panel — Decomposition into eigenpolygons

**Header.** The panel title "Decomposition into eigenpolygons".

**Colour bar.** A horizontal bar with min/max labels and a note line underneath. The scale is shared by all ten plots and expressed in units of cell area; $M$ is the largest absolute normalised value across the visible plots. For the signed measure the bar runs blue → gray → orange over $[-M, +M]$; for the unsigned and total-variation measures only the positive half, gray → orange over $[0, M]$, is used. Values are mapped with a power law (γ = ½).

**Checkbox — "Show basis polygons instead of eigenpolygons".** Unchecked by default. Toggling it animates every outline between $E_j$ and $F_j$ over 0.9 s (see §3.6) and changes the plot titles. While the basis polygon is shown, $c_j$ is marked by a red point. Cell colours always show the measure of $E_j$.

**Drop-down — "Alternative sense of measure".** Signed measure (default), Unsigned measure, Total-variation measure. It applies to the plots, the colour bar, the left-panel tooltip and every coverage table.

**Grid of plots (2 × 5, $j = 0, \ldots, 9$).** Each slot has a coloured top border (Tableau10, by $j$) and contains:

- a title: "Eigenpolygon $E_j$", or "Basis Polygon $F_j$" when the checkbox is on;
- a plot of the unit disc with the lattice, cells painted by the selected measure of $E_j$, and the polygon outline as a thin black dashed line whose dashes march in the direction of travel;
- degenerate cases: $E_0$ (and $F_0$), and any $E_j$ with $c_j = 0$, are drawn as a dot; a badge "×$g$" in the top-right corner marks a polygon traced $g = \gcd(j, d) > 1$ times;
- a caption with $c_j$ in polar form, `r ∠ θ°` with $\theta \in (-180°, 180°]$, and, for $j \neq 1$, the ratio $\lvert c_j \rvert / \lvert c_1 \rvert$;
- a hover tooltip with the cell index and the measure of $E_j$.

Slots with $j \geq d$ are greyed out and labelled "absent".

**Zoom inside a plot.** Each plot zooms independently of the page: Ctrl/⌘ + wheel or a trackpad pinch zooms (0.25× to 50×), dragging pans, and double-clicking resets. The current zoom factor is shown in the slot's top-left corner. Line widths, dash lengths and marker sizes stay constant on screen. A plain wheel is not captured, so the page still scrolls over the grid.

**Coefficients — $\lvert c_j \rvert$ vs $j$.** A bar chart in the design of a scree plot, with bars coloured by slot, a hover tooltip, and a "Switch to log scale / Switch to linear scale" button with animated transitions. On the log scale, coefficients below $10^{-12}$ of the largest are drawn as faint stubs.

**Exercises.** A collapsible section with four exercises, each with a title, a prompt with numbered questions, and a collapsible hint. They lead the student through the three facts of §1.1 (see the README for their pedagogical arc).

**Numeric details.** A collapsible section, collapsed by default. Its header carries an **"Export tables"** button that works whether or not the section is open. Contents, in order:

1. A header block with $d$, nspd, na, $g$; $B$ and $B^*$; the radius vector $a$; the number of sectors and cells and the cell area; and four self-checks with PASS/FAIL (see §4.6).
2. The coverage matrix $A$ of $z$ under the selected measure — rows are annuli with the innermost at the bottom, columns are sectors, with row and column sums — followed by $\operatorname{rank}(A)$.
3. The coefficient table: $\lvert c_j \rvert$, $\arg c_j$ in degrees, and $\lvert c_j \rvert / \lvert c_1 \rvert$.
4. The total signed areas $A(E_j)$ in a single row, followed by the reconstruction error $A(z) - \sum_j A(E_j)$.
5. For every eigenpolygon that encloses area, the coverage matrix $A_j$ of $E_j$, followed by $\operatorname{rank}(A_j)$, the sum of its cells and, under the signed measure, a check that this sum equals $A(E_j)$. Tables for zero-area eigenpolygons ($E_0$, $E_{d/2}$ for even $d$, and any $E_j$ with $c_j = 0$) are omitted, and a line lists which ones.

**Export.** "Export tables" writes `eigenpolygon-tables-d<d>-<measure>.ods` with one worksheet per table: *Parameters* (header values and $\operatorname{rank}(A)$), *A of z*, *Coefficients c_j*, *Signed areas*, and one *A_j of E_j* sheet per non-degenerate eigenpolygon (with its rank in a final row). Numbers are written at full precision. The coverage sheets keep the on-screen layout.

---

## 3. Technical specification

### 3.1 Technology stack

- Plain HTML, CSS and JavaScript in a single file; no build step, no framework.
- **d3 v7** (jsDelivr CDN) for SVG rendering, drag, zoom, transitions and the bar chart.
- **SheetJS 0.18.5** (jsDelivr CDN) for writing the `.ods` file; used only by the export button.

### 3.2 File structure

```
index.html     # the applet: markup, styles and script
README.md      # overview, usage, background and references
SPEC.md        # this document
LICENSE        # CC BY-NC-SA 4.0 license text
```

### 3.3 State model

A single `state` object holds: `d`, `nspd`, `na`, `g`; the radius vector `a`; the selected measure `mode` (`signed`, `unsigned`, `tv`); the checkbox flag `basis`; and the chart flag `logScale`. Derived results are cached in module-level variables: `lastF` (Fourier results), `lastMz` (coverage matrix of $z$), `lastME` (coverage matrices of the $E_j$) and `lastCross` (the cross-check value). Each slot of the grid keeps a reference record (`slotRefs[j]`) with its SVG elements and its current zoom factor.

### 3.4 Data flow

```
 user input (drag / slider / drop-down / checkbox)
        │
        ▼
     state ──► computeFourier()  ──►  z, c_j, F_j, E_j, A(z), A(E_j), checks
                     │
                     ▼
              measureMatrix(P, mode)  for P = z, E_0 … E_{d−1}
              signedMeasureMatrix(P)  (cross-check only)
                     │
                     ▼
     ┌───────────────┼──────────────────┬──────────────────────┐
     ▼               ▼                  ▼                      ▼
 buildGrid() /   updateScreePlot()  renderNumericDetails()  left-panel tooltip
 updateGrid()
```

`update()` runs the whole pipeline. `buildGrid()` rebuilds the slot structure only when `d`, `nspd` or `na` change; otherwise `updateGrid()` changes cell colours, outline paths and captions in place, so the dash animation and the per-slot zoom survive dragging. Toggling the checkbox does not run the pipeline; it only starts the morph animation. Any call to `update()` stops a running morph.

### 3.5 Core algorithms

**Fourier decomposition** (`computeFourier`). Builds $z_k = a_k \zeta_k$, computes $c_j = \frac1d \sum_k z_k \omega^{-jk}$ directly ($O(d^2)$, $d \leq 10$), forms $F_j$ and $E_j = c_j F_j$, and evaluates the signed areas with the shoelace formula. It also returns the vertex reconstruction error and both sides of Parseval's identity.

**Fan triangles** (`fanTriangles`). Each edge $(P_k, P_{k+1})$ defines a fan triangle $T_k = (0, P_k, P_{k+1})$ with orientation $\operatorname{sgn}(T_k)$, an angular interval, and the polar equation $\rho(\varphi) = h / \cos(\varphi - \varphi_0)$ of its far edge, where $h$ is the distance from the origin to the edge line and $\varphi_0$ the angle of its foot point. Degenerate triangles (zero cross product: coincident vertices, edges through the origin, digons) are skipped. Almost everywhere, $w_P = \sum_k \operatorname{sgn}(T_k)\,\mathbf 1_{T_k}$.

**Signed measure, fan method** (`signedMeasureMatrix`). For each fan triangle and sector, the intersection is again a fan triangle; its area inside a disc of radius $r$ has the closed form $\tfrac12 h^2 [\tan(\varphi - \varphi_0)]$ along the edge plus $\tfrac12 r^2 \Delta\varphi$ along the arc (`fanDiscArea`). Annulus values are differences of consecutive disc values. Used only as an independent cross-check.

**All three measures, strip method** (`measureMatrix`). The angular range $[0, 2\pi)$ is cut at *breakpoints*: sector bounds, fan-triangle ends, each edge's foot-point angle $\varphi_0$, edge/circle crossings $\varphi_0 \pm \arccos(h/r)$, and edge/edge intersection angles. Between consecutive breakpoints the radial order of all active edges and lattice circles is fixed. At the midpoint angle the active edges are sorted by $\rho$; walking outward from the origin, the winding number starts at $\sum \operatorname{sgn}(T_k)$ and drops by $\operatorname{sgn}(T_k)$ at each edge. Each radial strip with constant winding number $w$ contributes $f(w)$ times the difference of two closed-form integrals ($\tfrac12 r^2 \Delta\varphi$ for a circle, $\tfrac12 h^2 [\tan]$ for an edge), with $f(w) = w$, $\mathbf 1[w \neq 0]$ or $\lvert w \rvert$ for the signed, unsigned and total-variation measures. The result is exact up to rounding; no arc is approximated by a polyline and nothing is sampled.

**Numerical rank** (`singularValues`, `numericalRank`). One-sided Jacobi SVD on the orientation with fewer columns; the rank counts singular values above $10^{-9}\,\sigma_{\max}$.

### 3.6 Animation and interaction

- **Directed outlines.** `stroke-dasharray: 2px 1.4px`, animated via `stroke-dashoffset` from 3.4px to 0 every 0.6 s, with `vector-effect: non-scaling-stroke` so width and dashes stay fixed on screen under zoom. Decreasing the offset moves the dashes forward along the path, i.e. in the order of the vertices.
- **Morph between $F_j$ and $E_j$** (`morph`, `drawOutline`). The outline at parameter $t \in [0, 1]$ is $s(t)\,F_j$ with $s(t) = \bigl((1-t) + t\lvert c_j \rvert\bigr)\, e^{\,i t \arg c_j}$, so the polygon scales and rotates continuously; $t$ follows a cubic in-out easing over 900 ms, driven by `d3.timer`.
- **Slot zoom.** `d3.zoom` on each slot SVG, with a filter that accepts the wheel only together with Ctrl/⌘, a gentler `wheelDelta` than d3's default, and double-click mapped to a reset transition. Dot and red-point radii are divided by the zoom factor.

### 3.7 Coordinate systems

- **Complex plane.** Vertices and polygons are complex numbers; the unit disc is the region of interest.
- **Left panel.** `viewBox 0 0 100 100`, centre (50, 50), unit radius 44; $y$ is flipped so that the imaginary axis points up.
- **Slots.** `viewBox 0 0 100 100`, centre (50, 50), unit radius 46.
- **Lattice indices.** Sector $s$ covers angles $[2\pi s/S, 2\pi(s+1)/S)$, counted counter-clockwise from $\zeta_0$; annulus $m$ covers radii $[\sqrt{m/n_a}, \sqrt{(m+1)/n_a})$, with $m = 0$ innermost. Tables list annuli top-down with $m = 0$ at the bottom.

### 3.8 Styling

The layout, font-size variables (`--fs-title`, `--fs-body`, `--fs-meta`, `--fs-small`), colour bar, slot cards, bar chart, Exercises and Numeric details sections reuse the design of the Shape-Rank-SVD applet. The diverging colour map interpolates between `#2166ac` (negative), `#f2f2f2` (zero) and `#d6604d` (positive).

---

## 4. Mathematical specification

### 4.1 Polygon and decomposition

With $\omega = e^{2\pi i/d}$, $\zeta_k = \omega^k$ and $a_k \in B^*$:

$$z = (a_0\zeta_0, \ldots, a_{d-1}\zeta_{d-1}), \qquad c_j = \tfrac1d \sum_{k=0}^{d-1} z_k\,\omega^{-jk} = \tfrac1d \sum_{k=0}^{d-1} a_k\,\omega^{k(1-j)},$$

$$F_j = (1, \omega^j, \ldots, \omega^{(d-1)j}), \qquad E_j = c_j F_j, \qquad z = \sum_{j=0}^{d-1} E_j.$$

$F_j$ is the regular polygon $\{d/j\}$, traced $\gcd(j, d)$ times; $F_0$ is a point and, for even $d$, $F_{d/2}$ is a segment traced back and forth. Because the $a_k$ are real, $c_1 = \bar a \in (0, 1]$, and every $\lvert c_j \rvert \leq \bar a$.

### 4.2 Signed area

$$A(P) = \tfrac12 \operatorname{Im} \sum_k \overline{P_k}\,P_{k+1}, \qquad A(E_j) = \tfrac d2\,\lvert c_j \rvert^2 \sin\!\frac{2\pi j}{d}, \qquad A(z) = \sum_j A(E_j).$$

The last identity holds because the cross terms between $E_j$ and $E_l$ carry the factor $\sum_k \omega^{(l-j)k} = 0$ for $j \neq l$.

### 4.3 Polar lattice

$S = d \cdot n_{spd}$ sectors and $n_a$ annuli with radii $\sqrt{m/n_a}$, so every cell has area $\lvert C \rvert = \pi / (S\,n_a)$.

### 4.4 Cell measures

For a closed polygon $P$ with winding number $w_P$ and a cell $C$:

| Measure | Definition |
|---|---|
| signed | $\mu_C(P) = \int_C w_P$ |
| unsigned | $\kappa_C(P) = \lvert C \cap \lbrace w_P \neq 0 \rbrace \rvert$ |
| total-variation | $\tau_C(P) = \int_C \lvert w_P \rvert$ |

Always $\lvert \mu_C \rvert \leq \tau_C$, $\kappa_C \leq \tau_C$ and $0 \leq \kappa_C \leq \lvert C \rvert$; the three coincide for $z$, which is simple and counter-clockwise. In general $\mu_C(z) \neq \sum_j \mu_C(E_j)$, and likewise for $\kappa$ and $\tau$; only $\sum_C \mu_C(z) = A(z) = \sum_j A(E_j)$ holds.

### 4.5 Coverage matrices and rank

$A[m][s] = \text{measure}_{C_{s,m}}(z)$ and $A_j[m][s] = \text{measure}_{C_{s,m}}(E_j)$, an $n_a \times S$ matrix each. Rank is numerical, at relative tolerance $10^{-9}$.

### 4.6 Self-checks and tolerances

| Check | Condition for PASS |
|---|---|
| total measure of $z$ equals $A(z)$ | $< 10^{-9}$ |
| vertex reconstruction $\max_k \lvert z_k - \sum_j (E_j)_k \rvert$ | $< 10^{-10}$ |
| Parseval $\sum_j \lvert c_j \rvert^2 = \tfrac1d \sum_k a_k^2$ | $< 10^{-10}$ |
| cross-check strip vs fan method, signed, over $z$ and all $E_j$ | $< 10^{-10}$ |
| area reconstruction $A(z) - \sum_j A(E_j)$ | $< 10^{-10}$ |
| per-$E_j$ sum of cells equals $A(E_j)$ (signed only) | $< 10^{-9}$ |

Other constants: values below $10^{-10}$ are displayed as 0; an edge within a relative $10^{-12}$ of a lattice circle is treated as tangent to it (§6).

---

## 5. Non-functional requirements

- **Performance.** A full recompute of all coverage matrices at the largest setting ($d = 10$, nspd = 4, na = 8: 320 cells × 11 polygons) takes about 8 ms in a desktop browser, so dragging stays smooth.
- **Browser support.** Current Chrome, Firefox, Safari and Edge. Uses `vector-effect`, CSS animations, `ResizeObserver` and `<details>`.
- **Offline use.** Requires network access for the two CDN libraries; the export fails gracefully, with a status message, if SheetJS did not load.
- **Accessibility.** Not specifically addressed: colour is the main channel for cell values, and the controls rely on a mouse or trackpad.

---

## 6. Known limitations and design trade-offs

1. **Radial polygons only.** Vertices move along their own axes, so $z$ is always simple, star-shaped about the origin and counter-clockwise. Self-intersecting and clockwise polygons appear only as eigenpolygons.
2. **Tangency tolerance.** Symmetric configurations often produce edges exactly tangent to a lattice circle (e.g. $d = 4$ with all ones and even $n_a$). The strip method therefore adds each edge's foot-point angle as a breakpoint and compares radii with a relative tolerance of $10^{-12}$; an earlier version without these safeguards lost whole cells in such cases. The cross-check line in Numeric details guards against regressions.
3. **Shared colour scale.** All plots share one scale in units of cell area, so small eigenpolygons look pale. The per-slot zoom and the basis-polygon view compensate for their small size, but not for their faint colours.
4. **Modifier key for zoom.** Zooming needs Ctrl/⌘ so that the grid does not capture ordinary page scrolling.
5. **Degenerate tables omitted.** Coverage tables are hidden only for zero-area eigenpolygons; multiply traced polygons such as $E_2$ for $d = 6$ keep theirs.
6. **No $\sum_j A_j$ table.** The applet deliberately leaves summing the eigenpolygon tables to the student (via the export), because discovering that the sum differs from $A$ is the point of Exercise 3.

---

## 7. Extension points

Ordered roughly by implementation cost:

1. A toggle that shows $\sum_j A_j$ and the difference $A - \sum_j A_j$ next to the coverage matrix of $z$.
2. A per-slot colour normalisation option, as in Shape-Rank-SVD.
3. URL-encoded state, so that a configuration can be shared as a link.
4. Free vertices (general $d$-gons, not only radial ones), which would let $z$ itself self-intersect.
5. Touch-optimised dragging and zooming for tablets.

---

## 8. Change log

| Version | Date | Notes |
|---|---|---|
| 1.0 | 2026-11-12 | First public version. |
