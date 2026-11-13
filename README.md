# Shape-Rank-PDN

Did you know that every polygon is a sum of regular polygons? Not only the familiar convex ones, but also stars, and a few degenerate shapes such as a single point or a segment traversed back and forth. **Shape-Rank-PDN** is an interactive applet that makes this decomposition visible. Drag the vertices of a polygon, watch it split into its *eigenpolygons*, and find out which properties of the shape survive the split and which do not.

Why should anyone care about this decomposition? Interpretability has become a mandatory feature of AI applications in many domains. A diagram that *faithfully* depicts what a learning algorithm does promotes its interpretability, offering a route to understanding that avoids post-hoc rationalisations. [We explore such diagrams in our research](https://github.com/andreplima/polygrid). To ensure faithfulness, we use a diagram as a template to extract a feature matrix $S$ from the original data $X$, and train a linear model $SW = Y$ to solve the task at hand. A polygon drawn from a row of $X$ is an obvious candidate for such a template, and its Fourier decomposition is an obvious candidate for explaining what the features measure. This applet shows how far that explanation goes, and where it stops: the decomposition is exact for the polygon and for its total area, but it does **not** carry over to the area that the polygon leaves in each cell of a lattice. That gap matters, because the cell areas are the features.

The name refers to the Petr–Douglas–Neumann theorem, a classical result of polygon geometry whose standard proof rests on this decomposition.

**Live applet:** https://andreplima.github.io/shape-rank-pdn/

---

## What the applet shows

Suppose your data matrix $X \in (0,1]^{m \times d}$ has $3 \leq d \leq 10$ columns. Select an arbitrary row and represent its $d$ values $a_0, \ldots, a_{d-1}$ by a polygon instead of a point, as in a radar chart: value $a_k$ becomes a vertex on the $k$-th of $d$ equally spaced axes. Writing $\zeta_k = \omega^k$ with $\omega = e^{2\pi i/d}$ for the $d$-th roots of unity, the polygon is the vector of complex vertices

$$z = (a_0\zeta_0, \ldots, a_{d-1}\zeta_{d-1}).$$

In the applet the values are quantised to $B^* = \{1/g, 2/g, \ldots, 1\}$, where $g$ is the *granularity*.

**Eigenpolygons.** The discrete Fourier transform splits $z$ into $d$ additive components,

$$z = \sum_{j=0}^{d-1} E_j, \qquad E_j = c_j F_j, \qquad F_j = (1, \omega^j, \omega^{2j}, \ldots, \omega^{(d-1)j}), \qquad c_j = \tfrac1d \sum_{k} z_k\,\omega^{-jk}.$$

Each *basis polygon* $F_j$ is a regular polygon inscribed in the unit circle: convex for $j = 1$ and $j = d-1$, a star for most other $j$, and degenerate in two cases, namely a point for $j = 0$ and a segment traversed back and forth for $j = d/2$. When $\gcd(j, d) > 1$ it traces a smaller regular polygon several times. The *eigenpolygon* $E_j$ is $F_j$ scaled by $|c_j|$ and rotated by $\arg c_j$.

**Polar lattice and coverage matrices.** The unit disc is divided into $d \cdot n_{spd}$ sectors and $n_a$ annuli, with radii $\sqrt{m/n_a}$, so that all cells have the same area. The **coverage matrix** of a polygon $P$ records, for each cell $C$, how much of $P$ falls in it. Because eigenpolygons can be stars that wind around a point more than once, or run clockwise, the applet offers three senses of measure, all based on the winding number $w_P$:

| Sense | Cell value | Meaning |
|---|---|---|
| Signed | $\int_C w_P$ | area counted with multiplicity and orientation |
| Unsigned | area of $C \cap \lbrace w_P \neq 0 \rbrace$ | area the polygon visibly covers |
| Total-variation | $\int_C \lvert w_P \rvert$ | area counted with multiplicity, ignoring orientation |

For $z$ itself, which never crosses itself and always runs counter-clockwise, the three senses coincide.

**The pedagogical arc** that the applet supports in a classroom has three stages:

1. **Every polygon is a sum of regular polygons.** The decomposition $z = \sum_j E_j$ is exact, vertex by vertex, and its terms are regular, star or degenerate polygons.
2. **Total area decomposes too.** Signed area is a quadratic quantity, yet the cross terms between different eigenpolygons all cancel, so $A(z) = \sum_j A(E_j)$, with $A(E_j) = \tfrac d2 \lvert c_j \rvert^2 \sin(2\pi j / d)$.
3. **Cell areas do not decompose.** The coverage matrix of $z$ is *not* the sum of the coverage matrices of the $E_j$: not cell by cell, not when aggregated by annulus or by sector, and, outside the signed sense, not even in total. The equality in stage 2 is a global fact about the whole plane, and it does not localise to the lattice.

Four built-in exercises, in the **Exercises** section of the applet, walk students through these three stages, each with a hint that can be revealed once the student has made a guess.

## How to use it

Open the live URL above, or clone and open `index.html` directly in a browser.

**Left panel: the polygon and the lattice.**

- **Drag** any blue vertex along its own axis. On release it snaps to the nearest value in $B^*$. Hovering over a cell shows its measure for $z$.
- **Sliders** set $d$ (3 to 10 vertices), $n_{spd}$ (1 to 4 sectors per wedge between adjacent axes), $n_a$ (1 to 8 annuli) and the granularity $g$ (2 to 10). Changing $d$ or $g$ resets the polygon to the regular one, with all $a_k = 1$.
- **Reset** returns every slider and the polygon to the defaults.

**Right panel: the decomposition.**

- The **grid of ten plots** shows $E_0, \ldots, E_{d-1}$. Cells are coloured by the selected measure of each $E_j$, on a scale shared by all plots and expressed in units of cell area. Outlines are animated dashed lines whose dashes march in the direction of travel, so counter-clockwise and clockwise polygons can be told apart. A badge such as "×2" marks a polygon traced more than once. Below each plot are $c_j$ in polar form and the ratio $\lvert c_j \rvert / \lvert c_1 \rvert$.
- **Show basis polygons instead of eigenpolygons** morphs each outline into $F_j$ and back, and marks $c_j$ with a red point. Cell colours keep showing $E_j$.
- **Alternative sense of measure** switches between signed, unsigned and total-variation measure.
- **Zoom inside a plot** with Ctrl (⌘ on a Mac) and the mouse wheel, or a trackpad pinch. Drag to pan and double-click to reset. This helps with eigenpolygons whose coefficient is small.
- **Coefficients** plots $\lvert c_j \rvert$ against $j$, on a linear or logarithmic scale.
- **Numeric details** (collapsed by default) lists the parameters, $B$ and $B^*$, the area $A(z)$ and four self-checks. It then shows the coverage matrix of $z$ with its rank, the coefficient table, the signed areas $A(E_j)$ with the reconstruction error $A(z) - \sum_j A(E_j)$, and the coverage matrix and rank of every eigenpolygon that encloses area.
- **Export tables** saves every table in Numeric details to an `.ods` spreadsheet, one table per sheet. Several exercises ask you to add tables together; the spreadsheet is the easiest place to do it.

## Background and references

The decomposition of a polygon into regular polygons goes back to Darboux (1878), who used what we now call the finite Fourier transform to study iterated midpoint polygons. Schoenberg (1950) stated it explicitly, and Bachmann and Schmidt (1970) and Davis (1979) developed it systematically. The name "eigenpolygon" comes from computer graphics; DeRose and Catmull proposed "basis polygons" instead, as reported by Smith (1998).

- G. Darboux (1878). Sur un problème de géométrie élémentaire. *Bulletin des Sciences Mathématiques et Astronomiques*, 2nd series, 2(1), p. 298 ff.
- I. J. Schoenberg (1950). The finite Fourier series and elementary geometry. *The American Mathematical Monthly*, 57(6), 390–404.
- F. Bachmann and E. Schmidt (1970). *n-Ecke*. Bibliographisches Institut, Mannheim. English translation: *n-Gons*, University of Toronto Press, 1975.
- P. J. Davis (1979). *Circulant Matrices*. Wiley, New York.
- A. R. Smith (1998). Eigenpolygon decomposition of polygons. Microsoft Technical Memo 19.

## Running locally

Clone the repo and open `index.html` in a browser. No server is required.

```bash
git clone https://github.com/andreplima/shape-rank-pdn.git
cd shape-rank-pdn
open index.html   # or xdg-open index.html on Linux
```

The applet needs network access for its two CDN dependencies: d3 and SheetJS, which is used only for exporting tables. For a fully offline version, download the two library files from the CDN URLs in the `<script>` tags at the top of `index.html`, host them alongside `index.html`, and update the paths.

## License

This applet is released under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA 4.0). See the [LICENSE](LICENSE) file for the full legal text.

In plain terms, you are free to:

- **Share** — copy and redistribute the applet in any medium or format;
- **Adapt** — remix, transform, and build on it (fix bugs, add features, translate the exercises, repurpose it for your own classroom);

under the following terms:

- **Attribution** — you must give appropriate credit, link back to this repository, and indicate if changes were made;
- **NonCommercial** — you may not use the applet for primarily commercial purposes;
- **ShareAlike** — if you adapt the applet, you must distribute your version under the same license.

Issues and pull requests are welcome.
