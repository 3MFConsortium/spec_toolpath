**Title:** Simplify modifier scale attributes: Drop `e`–`h` on geometry elements in favor of `e1`- `h1` with optional`e2` - `h2`

### Summary

Currently, modifier channels `e`, `f`, `g`, and `h` define three attributes per channel on segment geometry elements (`<point>`, `<hatch>`, `<point3d>`, `<point6d>`):

* **Constant scale:** `e`, `f`, `g`, `h`
* **Linear / nonlinear ramp scale:** `e1`/`e2`, `f1`/`f2`, `g1`/`g2`, `h1`/`h2`

This proposal removes the standalone constant attributes (`e`, `f`, `g`, `h`) from segment geometry elements while ​**retaining `e`–`h` exclusively on `<sub>` support points**​, and makes the end-of-segment attributes (`e2`, `f2`, `g2`, `h2`) optional. When only `*1` is specified (e.g. `e1`, `f1`,`g1`,`h1`), the scale factor applies uniformly across the segment; when both `*1`and`*2`are specified, the scale factor interpolates linearly from`*1`to`*2`(and through any`<sub>`control points when`type="nonlinear"`).

### Rationale

1. **Eliminates Unenforceable Co-Occurrence Rules in XSD 1.0:** XSD 1.0 cannot natively enforce mutual exclusivity between `e` and `e1`/`e2`, nor can it enforce that `e1` and `e2` always appear as a pair. Currently, invalid combinations (e.g., `<point e="0.5" e1="0.2" />` or `<point e1="0.2" />`) are only restricted in prose and pass schema validation. Removing `e`–`h` from geometry elements and making `*1` and `*2` independently optional eliminates invalid attribute states at the XSD level.
2. **Established Precedent in XML and Web Standards:** Using a single value to represent a uniform/constant state and an optional second value to define a ramp or dual-axis behavior is a well-established pattern in W3C and industry specifications—for example, SVG's `stdDeviation` (one value applies uniformly to X and Y; two values apply independently) and `<linearGradient>` stops, CSS shorthand properties (`gap`, `margin`, `padding`), and OpenAPI/JSON telemetry schemas where a scalar `v1` holds constant unless an optional `v2` target is provided.
3. **Preserves Clean Separation Between Spans and Interior Support Points (`<sub>`):** Geometry elements (`<hatch>`, `<point>`, `<point3d>`, `<point6d>`) represent spans with a start ($t = 0$) and end ($t = 1$), making `*1`/`*2` pairs natural. Conversely, `<sub>` elements represent single interior parametric positions $t \in (0, 1)$, so retaining single scalar `e`–`h` attributes exclusively on `<sub>` keeps `<sub>` strictly single-valued.
4. **Explicit Fallback & Continuity Semantics:** Every segment modifier reduces to a single interpolation model from start (`*1`) to end (`*2`):
   * **When `*2` is omitted:** `*2` defaults to `*1` (constant scale across the segment).
   * **When `*1` is omitted:**
     * For **discontiguous** segment objects (e.g., `<hatch>`), `*1` defaults to `1.0` (nominal profile value).
     * For **contiguous** segment objects (e.g., `<polyline>`, `<polyline3d>`, `<polyline6d>`), `*1` inherits the effective end value of the immediately preceding vertex (`*2`, or `*1` if `*2` was omitted), ensuring $C^0$ continuity across corners. On the first vertex of a polyline (or if no prior value exists), `*1` defaults to `1.0`.
5. **Reduced Binary Encoding Overhead:** Dropping 4 attributes (`e`, `f`, `g`, `h`) from geometry elements reduces the per-point attribute set from 12 modifier fields down to 8, saving 4 presence-bit flags in binary layer encodings.

### Proposed Changes

* **XSD Schema (`tp.xsd`):**
  * Remove attributes `e`, `f`, `g`, and `h` from `<point>`, `<hatch>`, `<point3d>`, and `<point6d>`.
  * Retain `e`, `f`, `g`, and `h` on `<sub>`.
  * Keep `e1`, `e2`, `f1`, `f2`, `g1`, `g2`, `h1`, and `h2` with `use="optional"` on geometry elements.
* **Specification Text (Part I §4.2 & Part II §1.3–1.4):**
  * Document the explicit fallback rules for omitted `*1` (discontiguous vs. contiguous primitives) and omitted `*2` (defaults to `*1`).
  * Update `ST_ModifierType` producer rules:
    * `type="constant"`: Only `*1` attributes (`e1`–`h1`) are permitted; `*2` attributes and `<sub>` children are prohibited.
    * `type="linear"`: Both `*1` and `*2` attributes are permitted; `<sub>` children are prohibited.
    * `type="nonlinear"`: `*1`, `*2`, and `<sub>` children (carrying `t` and single scalar `e`–`h`) are all permitted.
  * Update XML examples to replace `e`–`h` with `e1`–`h1` on `<hatch>` and `<point*>` elements.

