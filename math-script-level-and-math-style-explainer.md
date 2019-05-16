# `math-script-level` and `math-style` explainer

This document proposes two new CSS properties `math-style` and
`math-script-level` controlling how the `font-size` evolves inside
mathematical constructions. More generally `math-style` may be used to indicate
to implementations of math layout whether
[logical height](https://drafts.csswg.org/css-writing-modes-4/#extent) should
be minimized. For further discussions
on this proposal see
[math-script-level and math-style comments](math-script-level-and-math-style-comments.md).

## Examples

* `<div class="my-underover"><div>Base</div><div>Overscript</div><div>Underscript</div></div>`
  and `.my-underover > :not(:first-child) { math-script-level: add(1); }` would
  scale down the font size in under and over scripts.

* In the previous example, one could do
  `<div style="math-script-level: 0;">Overscript</div>` to reset the font-size
  in the overscript back to the initial base size.

* `<div class="my-fraction"><div>Numerator</div><div>Denominator</div></div>`
  and `.my-fraction > * { math-script-level: auto; }` would
  scale down the font size in the numerator or denominator, depending on the
  actual `math-style` on the the fraction.

* The LaTeX formula `$$A^{A^A} + \sqrt[A]{A} + \frac{A+\frac{A}{A}}{A}$$`
  uses text of different sizes.
  One could style equivalent HTML nodes with the
  `font-size` property but it could be more convenient to set the
  `math-script-level` (base, scripts, numerator/denominator) and
  `math-style` values (initially 'display' mode but 'inline' mode in inner
  constructions).
  ![$$A^{A^A} + \sqrt[A]{A} + \frac{A+\frac{A}{A}}{A}$$](/math-script-level-and-math-style-latex-example.png)

* Polyfills and native implementations can emulate MathML behavior such as
  `<mstyle displaystyle="true">...</mstyle>`,
  `<mstyle scriptlevel="2">...</mstyle>` or `<mstyle scriptlevel="+3">...</mstyle>` by mapping to `math-style: display`, `math-script-level: 2;` and
  `math-script-level: add(3);` respectively.

## Rationale

* Mathematical formulas are made of several nested constructions. Readers expect
  that each time you go deeper in the expressions (e.g. enter a superscript)
  text will be automatically scaled down. This is can be achieved by
  adding relative or absolute `font-size` rules on each
  relevant node, but that's a bit tedious and error-prone.

* The actual rules for the text size change are a bit complex, and involve
  concepts similar to `math-script-level` (how deep we are in the expression)
  and `math-style` (whether we try to minimize
  [logical height](https://drafts.csswg.org/css-writing-modes-4/#extent)).
  In the TeX example
  `$$A^{A^A} + \sqrt[A]{A} + \frac{A+\frac{A}{A}}{A}$$`,
  the `A` letter is scaled down when entering the superscripts but even
  faster when entering a root's prescript. Also it is scaled down when entering
  the inner fraction but not when entering the outer one. These cases on a
  basic example suggest that a standardized approach would be important to
  ensure interoperability.

* More generally, mathematical typesetting follows rules to decide whether
  math layout should try and minimize the
  [logical height](https://drafts.csswg.org/css-writing-modes-4/#extent) and
  this decision can change when going deeper in the expressions or by user
  request. In the previous example this affects whether we increase the
  `math-script-level` when entering fractions but there are more of them
  such as changing the layout of under/over scripts,
  reducing spacing or fraction's
  line thickness, etc Again, it's handy to use CSS for the inheritance and
  overriding of this `math-style` value even if most of its effect happens
  in the layout and painting phase.

* `math-script-level` and `math-style` are purely stylistic features so it makes
  sense to have them integrated in the style system.

* These properties intend to implement the following MathML attributes:
  `display`, `displaystyle` and `scriptlevel` attributes. Again, there are
  complex interactions between these attributes, MathML elements and `font-size`. Hence
  relying on native CSS properties seems the proper way to implement it.

* These properties can be used to emulate TeX's `\displaystyle`, `\textstyle`,
 `\scriptstyle`, and `\scriptscriptstyle` modes, they correspond to
  `math-style` and `math-script-level` as "display" and 0,
  "inline" and 0, "inline" and 1, and inline and 2, respectively. These are
  important use cases requested by math people.

* OpenType math fonts have values about how the script should scale down when
  going from script level 0 to script level 1 (`scriptPercentScaleDown`) or 2
  (`scriptScriptPercentScaleDown`). Although this propopal simply uses constant
  values, in theory the evolution of font-size could depend on these
  parameters. If they are considered in the future, having actual script level
  values would help implementers.

## Status in Web Engines

* The `displaystyle` and `scriptlevel` attributes are implemented in Gecko and
  Stylo via an internal CSS property. Actually, more features are implemented
  and maybe this is an opportunity to simplify things.
* The `displaystyle` attribute is implemented in WebKit using some custom
  CSS-like inheritance. It could be rewritten to rely on `math-style` instead.
  The default `scriptlevel` behavior is implemented but does take into account
  user-specified attribute and does not interact with `displaystyle`.
  Relying on `math-script-level` would allow a cleaner and more complete
  implementation.
* `displaystyle` and `scriptlevel` attributes are tested in MathML WPT, Gecko
  and WebKit tests. Having these separate properties could allow to do pure CSS
  tests related to the interaction with `font-size`.

## Proposal

### CSS `math-style` property

<table>
  <tbody>
    <tr><th>Name:</th><td>'math-style'</td></tr>
    <tr><th>Value:</th><td>display | inline</td></tr>
    <tr><th>Initial:</th><td>inline</td></tr>
    <tr><th>Applies to:</th><td>All elements</td></tr>
    <tr><th>Inherited:</th><td>yes</td></tr>
    <tr><th>Percentages:</th><td>n/a</td></tr>
    <tr><th>Media:</th><td>visual</td></tr>
    <tr><th>Computed value:</th><td>specified keyword</td></tr>
    <tr><th>Canonical order:</th><td>n/a</td></tr>
    <tr><th>Animation type:</th><td>not animatable</td></tr>
  </tbody>
</table>

If the value of `math-style` is 'inline', the math layout on descendants
should try to minimize the
[logical height](https://drafts.csswg.org/css-writing-modes-4/#extent).
This includes how `font-size` is changed when `math-script-level` is 'auto'
as well
miscelleanous layout rules described for the `displaystyle` attribute of MathML.
If the value of `math-style` is 'display', the math layout should not take
such constraints into consideration.

### CSS `math-script-level` property

<table>
  <tbody>
    <tr><th>Name:</th><td>math-script-level'</td></tr>
    <tr><th>Value:</th><td>auto | add(&lt;integer&gt;) | &lt;integer&gt;</td></tr>
    <tr><th>Initial:</th><td>0</td></tr>
    <tr><th>Applies to:</th><td>All elements</td></tr>
    <tr><th>Inherited:</th><td>yes</td></tr>
    <tr><th>Percentages:</th><td>n/a</td></tr>
    <tr><th>Media:</th><td>visual</td></tr>
    <tr><th>Computed value:</th><td>see individual properties</td></tr>
    <tr><th>Canonical order:</th><td>n/a</td></tr>
    <tr><th>Animation type:</th><td>not animatable</td></tr>
  </tbody>
</table>

If the specified value of `math-script-level` is 'auto' and the inherited
value of `math-style` is 'display' then the computed value of
`math-script-level` is the inherited value.

If the specified value of `math-script-level` is 'auto' and the inherited
value of `math-style` is 'inline' then the computed value of
`math-script-level` is the inherited value plus one.

If the specified value of `math-script-level` is of the form
'add(&lt;integer&gt;)' then
the computed value of `math-script-level` is the inherited value
plus the specified integer.

If the specified value of `math-script-level` is of the form '&lt;integer&gt;'
then the computed value of `math-script-level` is set to the specified integer.

If `font-size` is specified or if the specified value of `math-script-level`
is `initial` then `math-script-level` does not affect the computed value of
`font-size`.
Otherwise, the computed value of `font-size` is obtained by multiplying the
inherited value of `font-size` by a nonzero scale factor calculated by the
following procedure:

1. Let A be the inherited `math-script-level`, B the computed
  `math-script-level`, C be 0.71 and S be 1.0
  * If A = B then return S.
  * If B < A, swap A and B and set `InvertScaleFactor` to true.
  * Otherwise B > A and set `InvertScaleFactor` to false.
2. Let E be B - A > 0.
3. If the inherited font has an OpenType MATH table:
  * Read `scriptPercentScaleDown` and fallback to C if the MathVariants table is absent or provides a null value.
  * Read `scriptScriptPercentScaleDown` and fallback to C<sup>2</sup> if the MathVariants table is absent or provides a null value.
  * If A ≤ 0 and B ≥ 2 then multiply S by `scriptScriptPercentScaleDown` and
    decrement E by 2.
  * Otherwise if A > 0 and B ≥ 2 then multiply S by
    `scriptScriptPercentScaleDown` / `scriptPercentScaleDown` and
    decrement E by 1.
  * Otherwise if A ≤ 0 and B < 2 then multiply S by `scriptPercentScaleDown`
    and decrement E by 1.
4. Multiply S by C<sup>E</sup>
5. Return S if `InvertScaleFactor` is false and 1/S otherwise.

The clamping of `font-size` implied by `font-min-size` and `font-max-size` must
apply after the change due to `math-script-level`.

### Native implementations of `display`, `displaystyle` and `scriptlevel`

The proposal allows to partially implement the `display`, `displaystyle` and
`scriptlevel` attributes as follows:

* Map `scriptlevel="+U"` to 'math-script-level: add(U)' (where U is an [unsigned integer](https://www.w3.org/Math/draft-spec/chapter2.html#type.unsigned-integer)).
* Map `scriptlevel="-U"` to 'math-script-level: add(-U)' (where U is an [unsigned integer](https://www.w3.org/Math/draft-spec/chapter2.html#type.unsigned-integer)).
* Map `scriptlevel="U"` to 'math-script-level: U' (where U is an [unsigned integer](https://www.w3.org/Math/draft-spec/chapter2.html#type.unsigned-integer)).

Then add rules equivalent to the following user agent stylesheet for MathML.
Note that some rendering engines do not allow universal selectors in their
user agent stylesheets and so such rules must be expanded to list all possible
MathML elements.

<pre>
@namespace url(http://www.w3.org/1998/Math/MathML);

math {
  math-style: inline;
  math-script-level: initial;
}
math[display="block"] {
  math-style: display;
}
math[display="inline"] {
  math-style: inline;
}
math[displaystyle="false"] {
  math-style: inline;
}
math[displaystyle="true"] {
  math-style: display;
}
mtable {
  math-style: inline;
}
mtable[displaystyle="true"] {
  math-style: display;
}
mstyle[displaystyle="false"] {
  math-style: inline;
}
mstyle[displaystyle="true"] {
  math-style: display;
}
mfrac > * {
  math-script-level: auto;
  math-style: inline;
}
mroot > :not(:first-child) {
  math-script-level: add(2);
  math-style: inline;
}
msub > :not(:first-child),
msup > :not(:first-child),
msubsup > :not(:first-child),
mmultiscripts > :not(:first-child),
munder > :not(:first-child),
mover > :not(:first-child),
munderover > :not(:first-child) {
  math-script-level: add(1);
  math-style: inline;
}
</pre>
