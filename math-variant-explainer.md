# `math-variant` explainer

This document proposes a new CSS property `math-variant` similar to
[text-transform](https://drafts.csswg.org/css-text-3/#text-transform)
in order to map alphanumeric text to equivalent
[Mathematical Alphanumeric Symbols](https://en.wikipedia.org/wiki/Mathematical_Alphanumeric_Symbols). It will help implementers of MathML (or more generally of
tools to generate or display math content) by taking care of inheritance,
character mapping and miscellaneous other use cases. For further discussions
on this proposal see [math-variant comments](math-variant-comments.md).

## Examples

* `<span style="math-variant: fraktur">A</span>` would render the same as
  `<span>𝔄</span>` or `<span>&#x1D504;</span>`
  (U+1D504 MATHEMATICAL FRAKTUR CAPITAL A).

* The LaTeX formula
  `\exp{\mathbf{A}} \in {{\mathfrak{gl}}_n\left(\mathbb R\right)}`
  is written with the exponential function using "normal" variant,
  matrix A using "bold" variant, lie alegra using "fraktur" variant,
  integer n using "italic" variant and set of numbers R using "double-struck"
  variant. The `\math*` commands could just be mapped to
  equivalent HTML nodes, styled with the `math-variant` property.
  ![\exp{\mathbf{A}} \in {{\mathfrak{gl}}_n\left(\mathbb R\right)}](/math-variant-latex-example.png)

* A simple math formula like
  `<span style="math-variant: italic" style>x + y</span>` would have x, y in
   italic but not operator + (contrary to `font-style: italic`).

* `<span style="font-family: STIX Math; math-variant: bold">F</span>` would use
  the glyph for U+1D46D MATHEMATICAL BOLD ITALIC CAPITAL F contained in the
  STIX Math font. It would not try to look into the (non-existent) STIX Math
  bold font or emulate bold style from the F glyph of the STIX Math font.
  However, if STIX Math font is not available and none of the fonts used
  actually have a glyph for U+1D46D it will may this kind of fallback behavior,
  as it is done for `font-weight: bold`.

* Polyfills can emulate MathML behavior of
  `<mstyle mathvariant="double-struck">...</mstyle>`
  via `mstyle[mathvariant="double-struck"] { math-variant: double-struck; }`.
  More generally, native implementations could directly map the MathML
  `mathvariant` attribute to the CSS `math-variant` property.

## Rationale

* Mathematical formulae rely on mathematical alphanumeric symbols
  to provide specific meanings. For example, the Planck constant is denoted by
  an italic h, vectors in physics are often written as bold variables, set of
  numbers are represented by double-struck and Lie algebras by fraktur letters.
  One can rely on properties such as `font-style` or `font-weight` to render
  some of these symbols from basic alphanumeric characters but this is not
  possible for others like fraktur, double-struck, calligraphic, etc

* Unicode contains characters to describe this semantic information but they
  are difficult to typeset with standard keyboard. It might also be
  complicated for basic math parsers or renderers to properly inherit
  mathvariant and perform transformations from the original characters. For
  example in LaTeX,
  `$x + \mathfrak{\mathbb{\aleph+N} \sin \frac{2}{\alpha}}$` makes
  x italic, N double-struck, 2 fraktur and all the other characters are
  unchanged. Hence a CSS property similar to `text-transform` to easily perform
  transformation and inheritance would be helpful.

* Most OpenType math fonts have glyphs to represent these characters but don't
  have associated italic/bold/bold-italic fonts. For generic fonts, the
  situation is actually the opposite: italic/bold/bold-italic fonts are
  available but not the corresponding mathematical alphanumeric symbols. It
  is tedious for math content generators or renderers to handle these
  different cases and decide some reasonable fallback, so a standardized
  approach is needed.

* One important math typesetting convention is that single-character
  identifiers are italic by default. Again, since OpenType math fonts have
  italic mathematical alphanumeric symbols but no associated italic fonts,
  using `font-style: italic` does not provide the same visual rendering as
  the equivalent mathematical alphanumeric symbols.

* MathML defines a `mathvariant` attribute to facilitate performing these
  mathvariant transformations from the basic alphanumeric characters,
  some kind of inheritance (e.g. like `\mathfrak` above), automatic italic
  on single-character `<mi>` elements. Some fallback for things like italic
  or bold are also suggested.
  Mapping the `mathvariant` attribute to some CSS property would allow
  an implementation that is consistent with CSS inheritance or `text-transform`.

## Status in Web Engines

* `math-variant` is similar to `text-transform`, which is implemented in
  all engines.
* The `mathvariant` attribute is implemented in Gecko and Stylo via an internal
  CSS property.
  It could be aligned with this specification by exposing the property and
  doing other minor adjustments.
* The `mathvariant` attribute is partially implemented in WebKit using some custom
  CSS-like inheritance and changing text rendering in token
  elements but only works for single variable (e.g. does not handle
  `<mi mathvariant="fraktur">sl</mi>`). Relying on a CSS property would
  allow a cleaner and more complete implementation.
* There are [MathML tests for mathvariant](https://github.com/web-platform-tests/wpt/tree/master/mathml/relations/css-styling) that could easily be converted
  to test CSS `math-variant`. Gecko has layout tests for other edge cases
  described in this specification.

## Proposal

### CSS `math-variant` property

<table>
  <tbody>
    <tr><th>Name:</th><td>'math-variant'</td></tr>
    <tr><th>Value:</th><td>auto | normal | bold | italic | bold-italic | double-struck | bold-fraktur | script | bold-script | fraktur | sans-serif | bold-sans-serif | sans-serif-italic | sans-serif-bold-italic | monospace | initial | tailed | looped | stretched</td></tr>
    <tr><th>Initial:</th><td>auto</td></tr>
    <tr><th>Applies to:</th><td>All elements</td></tr>
    <tr><th>Inherited:</th><td>yes</td></tr>
    <tr><th>Percentages:</th><td>n/a</td></tr>
    <tr><th>Computed value:</th><td>specified keyword</td></tr>
    <tr><th>Canonical order:</th><td>n/a</td></tr>
    <tr><th>Animation type:</th><td>not animatable</td></tr>
  </tbody>
</table>

This property transforms the text rendered visually or exposed to assistive
technologies. It has no effect on the underlying content or during conversion
to plaintext and must not affect the content of a plain text copy & paste
operation.

The 'normal' value has no effects. If the text is the unique character of a text
node which is itself the unique child of an
[mi element](https://www.w3.org/Math/draft-spec/chapter3.html#presm.mi)
then 'auto' has the same effect as 'italic', otherwise it has no effects.

For all the other values, the transformed text is obtained by performing
conversion of each character that have a mapping in the
[math-variant tables](math-variant-tables.md) according
to the specified `math-variant` value.

If the text is the unique character of a text
node which is itself the unique child of a 
[MathML token](https://www.w3.org/Math/draft-spec/chapter3.html#presm.tokel)
with 'bold', 'italic' or 'bold-italic' value then
UAs must instead render the original character with the corresponding font
properties if the available fonts don't contain any glyph for the
transformed character. UAs may decide to apply similar fallback for text of
arbitrary length or location.

When handling text with `math-variant` and `text-transform` values, the
string conversion related to the latter value must be applied after the string
conversion related to the former value.
For example text in
`<span style="text-transform: capitalize; math-variant: bold">a</mtext>` would
render as U+1D41A MATHEMATICAL BOLD SMALL A rather than
U+1D400 MATHEMATICAL BOLD CAPITAL A.

### Native implementation of the `mathvariant` attribute

The `mathvariant` attribute can be implemented natively by relying on the
`math-variant` property. For any MathML element accepting the `mathvariant`
attribute that has a `mathvariant` attribute with a valid value, set the CSS
property on that element to the corresponding `math-variant` value.
