
# Shape-Rank-PDN

Did you know that every polygon is the sum of its eigenpolygons? **Shape-Rank-PDN** is an interactive applet [inspired by a Mathologer video](https://www.youtube.com/watch?v=WLAW5yz5O3E&ab_channel=Mathologer) that allows you to explore this result. Drag the vertices of a polygon, watch it split into its eigenpolygons, and find out which properties of the shape survive the split and which do not.

Why should anyone care about this result? People say that "a picture is worth a thousand words," which one takes to mean that visual aids can help communicate complex ideas efficiently. AI researchers have been facing the challenge of developing algorithms that can explain their decisions to end-users and developers. In fact, **interpretability** has become a mandatory feature of AI applications in critical domains. A diagram that _faithfully_ depicts what the algorithm does and can be understood by the user, promotes _the right kind of interpretability_: the one that offers a route to understanding that avoids post-hoc rationalisations. Moreover, at the end of the day, pictures, visual aids, and diagrams all live in 2D world, where polygons play a fundamental descriptive role. Know your bricks, build a castle.

[We explore such diagrams in our research](https://github.com/andreplima/polygrid). To ensure faithfulness, we extract a feature matrix $S$ from the original data $X$ based on how the latter "are drawn on a template diagram," and train a linear model $SW = Y$ to solve the task at hand. Since the rank of the feature matrix plays an important role in the solvability of the linear system, we should seek templates that can somehow extract a high-rank feature matrix. This applet was originally conceived to help us discuss the advantages and shortcomings that the decomposition into eigenpolygons could bring to our research, but we decided to share it for its value as a tool for disseminating this curious mathematical result. However, it must be said that the applet is limited to the analysis of radial polygons, which is the class of polygons that we explore in our research.

The name **Shape-Rank-PDN** refers to the Petr–Douglas–Neumann theorem, a classical result of polygon geometry whose standard proof rests on this decomposition.

**Live applet:** https://andreplima.github.io/shape-rank-pdn/

----------

## What the applet shows

Suppose your data matrix $X \in (0,1]^{m \times d}$ has $d \geq 3$ columns. Select an arbitrary row and represent its $d$ values $a_0, \ldots, a_{d-1}$ by a **radial polygon**, as in a radar chart: value $a_k$ becomes a vertex on the $k$-th of $d$ equally spaced axes. Writing $\zeta_k = \omega^k$ with $\omega = e^{2\pi i/d}$ for the $d$-th roots of unity, the polygon is the vector of complex vertices:

$$z = (a_0\zeta_0, \ldots, a_{d-1}\zeta_{d-1}).$$

In the applet the values for $a_k$ are quantised to $B^* = {1/g, 2/g, \ldots, 1}$, where $g$ is the _granularity_.

**Eigenpolygons.** The discrete Fourier transform splits $z$ into $d$ additive components,

$$z = \sum_{j=0}^{d-1} E_j, \quad E_j = c_j F_j, \quad c_j = \tfrac1d \sum_{k=0}^{d-1} a_k \omega^{k(1-j)}, \quad F_j = (1, \omega^j, \omega^{2j}, \ldots, \omega^{(d-1)j}).$$

Each _basis polygon_ $F_j$ is a regular polygon inscribed in the unit circle: convex for $j = 1$ and $j = d-1$, a star for most other $j$, and degenerate in two cases, namely a point for $j = 0$ and a segment traversed back and forth for $j = d/2$. When $\gcd(j, d) > 1$ it traces a smaller regular polygon multiple times. The _eigenpolygon_ $E_j$ is $F_j$ scaled by $|c_j|$ and rotated by $\arg c_j$.

**Polar lattice and coverage matrices.** The unit disc is divided into $n_s = d \cdot n_{spd}$ sectors ($n_{spd}$ per wedge between adjacent axes) and $n_a$ annuli, with radii $\sqrt{p/n_a}$ for $p = 0, \ldots, n_a$, so that all cells have the same area. The **coverage matrix** of an arbitrary polygon $P$ (e.g., $P = z$ or $P = E_j$) records, for each cell $C$, how much of $P$ falls in it. Because eigenpolygons can be stars that wind around a point more than once, or run clockwise, the applet offers three senses of measure, all based on the winding number $w_P$:


| Sense | Cell value | Meaning |
|---|---|---|
| Signed | $\int_C w_P$ | area counted with multiplicity and orientation |
| Unsigned | area of $C \cap \lbrace w_P \neq 0 \rbrace$ | area the polygon visibly covers |
| Total-variation | $\int_C \lvert w_P \rvert$ | area counted with multiplicity, ignoring orientation |

For $z$ itself, which never crosses itself and always runs counter-clockwise, the three senses coincide.

**The pedagogical arc** that the applet supports in a classroom has three stages:

1.  **Every polygon is a sum of eigenpolygons.** The decomposition $z = \sum_j E_j$ is exact, vertex by vertex, and its terms are regular, star or degenerate polygons.
2.  **Total area decomposes too.** Signed area is a quadratic quantity, yet the cross terms between different eigenpolygons all cancel, so $A(z) = A(\sum_j E_j) = \sum_j A(E_j)$.
3.  **Cell areas do not decompose.** The coverage matrix of $z$ is _not_ the sum of the coverage matrices of the $E_j$: not cell by cell, not when aggregated by annulus or by sector, and, outside the signed sense, not even in total. The equality $A(z) = \sum_j A(E_j)$ is a global fact about the whole plane, and it does not localise to the lattice: in general, $\int_C w_z \neq \sum_j \int_C w_{E_j}$.

Four built-in exercises, in the **Exercises** section of the applet, walk students through these three results, each with a hint that can be revealed once the student has made a guess.

## How to use it

Open the live URL above, or clone and open `index.html` directly in a browser.

**Left panel: the polygon and the lattice.**

-   **Drag** any blue vertex along its own axis. On release it snaps to the nearest value in $B^*$. Hovering over a cell shows its measure for $z$.
-   **Sliders** set $d$ (3 to 10 vertices), $n_{spd}$ (1 to 4 sectors per wedge between adjacent axes), $n_a$ (1 to 8 annuli) and the granularity $g$ (2 to 10). Changing $d$ or $g$ resets the polygon $z$ to the regular one, with all $a_k = 1$.
-   **Reset** returns every slider and the polygon to the defaults.

**Right panel: the decomposition.**

-   The **grid of ten plots** shows $E_0, \ldots, E_{d-1}$. Cells are coloured by the selected measure of each $E_j$, on a scale shared by all plots and expressed in units of cell area. Outlines are animated dashed lines whose dashes march in the direction of travel, so counter-clockwise and clockwise polygons can be told apart. A badge such as "×2" marks a polygon traced more than once. Below each plot are $c_j$ in polar form and the ratio $\lvert c_j \rvert / \lvert c_1 \rvert$.
-   **Show basis polygons instead of eigenpolygons** morphs each eigenpolygon into $F_j$ and back, and marks $c_j$ with a red point. Cell colours keep showing how $E_j$ covers the lattice.
-   **Alternative sense of measure** switches between signed, unsigned and total-variation measure.
-   **Zoom inside a plot** with Ctrl (⌘ on a Mac) and the mouse wheel, or a trackpad pinch. Drag to pan and double-click to reset. This helps with eigenpolygons whose coefficient is small.
-   **Coefficients** plots $\lvert c_j \rvert$ against $j$, on a linear or logarithmic scale.
-   **Numeric details** (collapsed by default) lists the parameters, $B$ and $B^*$, the area $A(z)$ and four self-checks. It then shows the coverage matrix of $z$ with its rank, the coefficient table, the signed areas $A(E_j)$ with the reconstruction error $A(z) - \sum_j A(E_j)$, and the coverage matrix and rank of every eigenpolygon that encloses area.
-   **Export tables** saves every table in Numeric details to an `.ods` spreadsheet, one table per sheet. Several exercises ask you to add tables together; the spreadsheet is the easiest place to do it.

## Background and references

Decomposing a polygon into simpler ones is an ancient idea, usually taken to mean **dissection**: a rectangle $R$ cut into two triangles $T_1$ and $T_2$ is the union of regions that tile it, $R = T_1 \cup T_2$, and its area is the sum of their areas by construction, $A(R)=\sum_i A(T_i)$. The decomposition used here is of a different kind. A polygon with $d$ vertices is treated as an ordered list of points $z = (z_0, z_1, \ldots, z_{d-1}) \in \mathbb{C}^d$, and it is written as a _vertex-wise sum_ of regular polygons $z = \sum_j E_j$. The summands $E_j$ do not tile the region bounded by $z$; they overlap, may run clockwise, and may extend beyond it. It is the positions of the vertices, not the regions, that add up. Because the summands are not a dissection, there is no reason for area to add up. And yet, remarkably, total signed area does: $A(z) = \sum_j A(E_j)$, even though the signed area that falls within a given region $C$ of the plane, in general, does not: $\int_C w_z \neq \sum_j \int_C w_{E_j}$.

This idea of representing a polygon by a **superposition** of regular polygons, $z = \sum_j E_j$, is much more recent; it developed along two parallel lines of work, both about transformations that build a new polygon from an old one, side by side.

### Iterated midpoint polygons

According to Hintikka and Sun [6], Darboux [2] used what is now called the finite Fourier transform to study polygons obtained by repeatedly joining the midpoints of the sides, and proved that they converge to their common centroid. Schoenberg [10] made the decomposition into regular polygons explicit. Berlekamp, Gilbert and Sinden [1], asking which polygons eventually have convex midpoint descendants, stated that "an arbitrary $N$-gon can be written as a sum of regular $N$-gons", counting self-intersecting, multiply traversed and degenerate polygons as regular, and called the components _eigenpolygons_, the earliest use of the term we have found.

### Placing triangles on the sides

The second line goes back to a classical result about triangles. If equilateral triangles are erected on the sides of any triangle, all outward or all inward, their centres form an equilateral triangle. The result is attributed, on doubtful grounds, to Napoleon [5]. Since the centre of an equilateral triangle erected on a side is the apex of an isosceles triangle with apex angle 120° on that side, the same theorem can be stated with isosceles triangles. It was in this latter form that Petr [8, 9] generalised it to $n$-gons: erecting isosceles triangles on the sides of an arbitrary $n$-gon and taking their apexes as the vertices of a new $n$-gon, $n-2$ such steps, with suitably chosen apex angles, turn any $n$-gon into a regular one.

### Unification

The two lines met in the work of Douglas [3, 4]. Prompted by earlier work on midpoint polygons, he rediscovered Petr's result and placed both constructions in a general theory of _linear polygon transformations_: linear maps on the list of vertices that commute with a cyclic shift, whose matrices are circulant. In this theory the midpoint step is simply the isosceles construction with a straight apex angle. Douglas did not write the decomposition down; instead he characterised regularity dually, by showing that the only intrinsic linear relations a polygon can satisfy are the vanishing of what are now its discrete Fourier coefficients, each expressing one "degree of regularity". Neumann [7], independently, wrote the decomposition down explicitly, borrowing from electrical engineering the name _symmetrical components_ for the regular, star and degenerate polygons that add up, vertex by vertex, to the original. He showed that each construction rotates and scales every component separately, so a suitable apex angle annihilates one component at a time. The result is now known as the Petr–Douglas–Neumann theorem, which gives this applet its name. Both kinds of construction were later treated within a single theory in the monographs of Bachmann and Schmidt (1970) and Davis (1979), the latter in the language of circulant matrices, whose eigenvectors are the eigenpolygons. The term _eigenpolygon_ emerged again from this second line of research, independently, in Smith [11], who reports that DeRose and Catmull preferred _basis polygons_, the name this applet uses for the $F_j$.

### References

1.  E. R. Berlekamp, E. N. Gilbert and F. W. Sinden (1965). [A polygon problem](https://www.jstor.org/stable/2313689). _The American Mathematical Monthly_, 72(3), 233–241.
2.  G. Darboux (1878). [Sur un problème de géométrie élémentaire](https://www.numdam.org/item/BSMA_1878_2_2_1_298_1/). _Bulletin des Sciences Mathématiques et Astronomiques_, 2nd series, 2(1), 298 ff.
3.  J. Douglas (1940). [Geometry of polygons in the complex plane](https://doi.org/10.1002/sapm194019193). _Journal of Mathematics and Physics_, 19, 93–130.
4.  J. Douglas (1940). [On linear polygon transformations](https://doi.org/10.1090/S0002-9904-1940-07259-3). _Bulletin of the American Mathematical Society_, 46(6), 551–560.
5.  B. Grünbaum (2012). [Is Napoleon's theorem _really_ Napoleon's theorem?](https://doi.org/10.4169/amer.math.monthly.119.06.495) _The American Mathematical Monthly_, 119(6), 495 ff.
6.  E. Hintikka and X. Sun (2016). [Convergence of sequences of polygons](https://doi.org/10.2140/involve.2016.9.751). _Involve_, 9(5), 751–764.
7.  B. H. Neumann (1941). [Some remarks on polygons](https://doi.org/10.1112/jlms/s1-16.4.230). _Journal of the London Mathematical Society_, s1-16(4), 230–245.
8.  K. Petr (1905). [O jedné větě pro mnohoúhelníky rovinné](https://www.dml.cz/bitstream/handle/10338.dmlcz/120936/CasPestMatFys_034-1905-2_8.pdf). _Časopis pro pěstování mathematiky a fysiky_, 34(2), 166–172.
9.  K. Petr (1908). Ein Satz über Vielecke. _Archiv der Mathematik und Physik_, 13, 29–31.
10.  I. J. Schoenberg (1950). The finite Fourier series and elementary geometry. _The American Mathematical Monthly_, 57(6), 390–404.
11.  A. R. Smith (1998). [Eigenpolygon decomposition of polygons](https://alvyray.com/Memos/CG/Microsoft/19_eigen.pdf). Microsoft Technical Memo 19.

### Further reading

-   F. Bachmann and E. Schmidt (1970). _n-Ecke_. Bibliographisches Institut, Mannheim. English translation: _n-Gons_, University of Toronto Press, 1975.
-   P. J. Davis (1979). _Circulant Matrices_. Wiley, New York.

## Running locally

Clone the repo and open `index.html` in a browser. No server is required.

```bash
git clone https://github.com/andreplima/shape-rank-pdn.git
cd shape-rank-pdn
open index.html   # or xdg-open index.html on Linux

```

The applet needs network access for its two CDN dependencies: d3 and SheetJS, which is used only for exporting tables. For a fully offline version, download the two library files from the CDN URLs in the `<script>` tags at the top of `index.html`, host them alongside `index.html`, and update the paths.

## License

This applet is released under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA 4.0). See the LICENSE file for the full legal text.

In plain terms, you are free to:

-   **Share** — copy and redistribute the applet in any medium or format;
-   **Adapt** — remix, transform, and build on it (fix bugs, add features, translate the exercises, repurpose it for your own classroom);

under the following terms:

-   **Attribution** — you must give appropriate credit, link back to this repository, and indicate if changes were made;
-   **NonCommercial** — you may not use the applet for primarily commercial purposes;
-   **ShareAlike** — if you adapt the applet, you must distribute your version under the same license.

Issues and pull requests are welcome.
