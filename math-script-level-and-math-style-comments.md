# `math-script-level` and `math-style` comments

## Script level increment in `munder`, `mover`, `munderover` scripts

When these elements are really drawn as under/over scripts then the script
level increment should not happen when `accentunder` or `accent` are specified:

<pre>
munder[accentunder="false"] > :nth-child(2),
mover[accent="false"] > :nth-child(2),
munderover[accentunder="false"] > :nth-child(2),
munderover[accent="false"] > :nth-child(3) {
  math-script-level: inherit;
}
</pre>

Unfortunately whether the elements are drawn with under/over scripts as well
as the default values of the `accentunder` and `accent` attributes have to be
determined from the MathML DOM and even using an
[operator dictionary](https://www.w3.org/Math/draft-spec/appendixc.html).
This is not possible to do in the CSS side. Maybe this feature could be removed
from MathML and the authors should explicitly indicate whether they want
to prevent increment in scriptlevel in this case.
Gecko implements it via pseudo-element set in the layout code 
but that's [causing some issues](https://bugzilla.mozilla.org/show_bug.cgi?id=1361766).

## `scriptsizemultiplier` and `scriptminsize` attributes

MathML also describes some `scriptsizemultiplier` and `scriptminsize` attributes
to respectively specify another value for the scale factor S (default is 0.71,
approximately one over square root of 2) and ensure
script level change cannot scale down `font-size` lower than a minimal value.
This is implemented in Gecko but that [makes things very complex](https://dxr.mozilla.org/mozilla-central/source/servo/components/style/properties/gecko.mako.rs#2481).

It's not clear whether the `scriptsizemultiplier` and `scriptminsize` attributes
are used a lot in practice so this MathML attributes could probably be removed.
Additionally, taking into account `scriptminsize` (even the default value of
8pt) means that for very deeply nested mathematical formulas, the `font-size`
will stop being scaled down in scripts etc. It's not clear whether that's
an important use case and whether that's better than reaching too small
̀font-size`. There are user features (e.g. zoom-in) to workaround that kind
of not frequent issues.

## OpenType MATH values: `scriptPercentScaleDown` and `scriptScriptPercentScaleDown`

OpenType math fonts have values about how the script should scale down when
going from script level 0 to script level 1 (`scriptPercentScaleDown`) or 2
(`scriptScriptPercentScaleDown`). That would suggest that a user-specified
`scriptsizemultiplier` is not really want, instead the one specified by the
font designer should be used.

This proposal chose a simple solution as it's not clear whether reading the
MATH table is a good idea or possible during the style resolution. Also
`scriptPercentScaleDown` and `scriptScriptPercentScaleDown` are not implemented
by any web engine right now ([Gecko bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1187682)). An algorithm could be:

* Use S = `scriptPercentScaleDown` when `math-script-level` goes from 0 to 1.
* Use S = `scriptScriptPercentScaleDown` divided by `scriptPercentScaleDown` when `math-script-level` goes from 1 to 2.
* Use S = 0.71 in other situation when `math-script-level` is incremented by 1.
* Generalize to values of Δ other than +1 by using product and inversion.

with adjustments when specified scales are zero and deciding whether to ignore
suggested values from the MATH table.
