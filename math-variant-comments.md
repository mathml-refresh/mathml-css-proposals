# `math-variant` comments

## Mixing with capitalize/uppercase/lowercase, full-width, full-size-kana

It could make sense to render something like
`<span style="text-transform: uppercase math-bold">a</mtext>`
as U+1D400 MATHEMATICAL BOLD CAPITAL A. However it's not clear
whether there is a clear use case to mix `math-...` values
with capitalize/uppercase/lowercase, full-width or full-size-kana. At least
that's not needed for MathML.

For the record, Gecko relies on a separate `math-variant` property and
string conversion related to `text-transform` is applied after the string
conversion related to `math-variant`.

## Handling of "math-bold", "math-italic" and "math-bold-italic" fallback

To render text, browsers split the rendering of text in multiple substrings
(e.g. linebreak fragments) and an implementation of `text-transform` may apply
to this individual substrings. Gecko applies the
"bold", "italic" and "bold-italic" font property unless at least one character
in the converted subtring can be rendered with available fonts. This
algorithm is not ideal as it depends on how the string is split in subtrings.

In practice, the most important math use case is MathML tokens with
single-character text (including the automatic italic on single-character
`<mi>`) so that's the only required fallback in this proposal. It can be
implemented in an interoperable way.

## Handling of whitespace

MathML asks to [collapse whitespace in tokens](https://www.w3.org/Math/draft-spec/chapter2.html#fund.collapse). This might in particular affect how one
determines whether `<mi>` has a single character. For example, Gecko treats
`<mi> x </mi>` as `<mi>x</mi>` and hence x is rendered italic. The way
MathML handles whitespace should probably be changed in future MathML
specification so that it is not necessary to collapse whitespace. Hence this
proposal ignores whitespace collapsing.
See [issue 28](https://github.com/mathml-refresh/mathml/issues/28].

## Conflicts with deprecated `fontstyle` and `fontweight` attributes

The `fontstyle` and `fontweight` attributes are [deprecated in MathML 3](https://www.w3.org/Math/draft-spec/chapter3.html#3.2.2.1).
In theory, they could have been implemented by applying the
corresponding CSS properties. Since `mathvariant` is supposed to take precedence
it must "revert" the corresponding font style on the text (this is done in Gecko
and Stylo). Support for these `fontstyle` and `fontweight` attributes should
probably just be removed and hence the CSS code simplified to match the present
proposal.
See [issue 5](https://github.com/mathml-refresh/mathml/issues/5)

## Semantics versus Styling

Mathematical alphanumeric symbols is supposed to convey some special semantic
and hence `mathvariant` is not strictly for stylistic purpose, so relying on CSS
might be questionable. Note however that:
* The distinction is not always obvious, for example putting single-character
  identifiers in italic
  is more a stylistic convention. Also MathML and Unicode currently
  don't distinguish between "script" and "calligraphic" (`\mathscr` and
  `\mathcal` in LaTeX) even if some people have requested it.
* Even for `text-transform` [it is arguable whether the original or transformed text should be exposed to assistive technology](https://groups.google.com/a/chromium.org/forum/#!msg/chromium-accessibility/enk1PBjEfRc/InWurOYUg0EJ).

This proposal does not try to be too strict about this semantics/styling
disticntion
and just focuses on a pragmatic approach to solve existing use cases via
CSS. In order to facilitate implementations, it aligns on `text-transform` and
in particular may contradict the MathML specification.

## Length of transformed text in UTF-16

Contrary to other `text-transform` values, many of the transformed characters
for `mathvariant` are in the Supplementary Multilingual Plane and hence require
surrogate pairs to be encoded in UTF-16. Implementers should take that into
account as that affects the length of the UTF-16 representation of the
transformed text.

## Handling of single-character `<mi>` for `text-transform: math-auto`

CSS resolver may
not have access to the DOM so the special handling of single-character
`<mi>` has to be handled somewhere else. In particular the computed value is
the value on single-character `<mi>` or returned by the
`getComputedStyle()` on such an element remains "math-auto".
