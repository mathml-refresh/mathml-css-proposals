# `math-script-level` and `math-style` comments

## Script level increment in `munder`, `mover`, `munderover` scripts

The current proposal does not handle the case when underover scripts are
"accents". See [issue 76](https://github.com/mathml-refresh/mathml/issues/76).

## `scriptsizemultiplier` and `scriptminsize` attributes

MathML also describes some `scriptsizemultiplier` and `scriptminsize` attributes
to respectively specify another value for the scale factor S (default is 0.71,
approximately one over square root of 2) and ensure
script level change cannot scale down `font-size` lower than a minimal value.
This is implemented in Gecko but that [makes things very complex](https://dxr.mozilla.org/mozilla-central/source/servo/components/style/properties/gecko.mako.rs#2481).

We should probably [remove these attributes](https://github.com/mathml-refresh/mathml/issues/1#issuecomment-474261094) from MathML Core.

## OpenType MATH values: `scriptPercentScaleDown` and `scriptScriptPercentScaleDown`

OpenType math fonts have values about how the script should scale down when
going from script level 0 to script level 1 (`scriptPercentScaleDown`) or 2
(`scriptScriptPercentScaleDown`). That would suggest that a user-specified
`scriptsizemultiplier` is not really want, instead the one specified by the
font designer should be used.

Note that TeX or Microsoft Office ignore other script level changes, which would
be equivalent to replacing 0.71 with 1 in the definition of S<sub>p,p+1</sub>.
The current definition tries to preserve MathML's special behavior.
