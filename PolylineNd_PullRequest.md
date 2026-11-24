**Title:** Unify polyline primitives: Rename `polyline`/`point` to `polyline2d`/`point2d` and replace `loop` with `closed` attribute

### Summary

Currently, planar and multi-axis polyline primitives have asymmetric naming and inconsistent support for closed contours:

* **Asymmetric naming:** `<polyline>` and `<point>` omit their dimensionality, whereas 3-axis and 6-axis equivalents are explicitly named `<polyline3d>`/`<point3d>` and `<polyline6d>`/`<point6d>`.
* **Planar-only closed contours:** `<loop>` only exists for 2D planar paths, leaving `<polyline3d>` and `<polyline6d>` with no native way to represent closed contours without duplicating start/end vertices.

This proposal renames `<polyline>` and `<point>` to `<polyline2d>` and `<point2d>`, and replaces `<loop>` with an optional `closed` boolean attribute (default `false`) across `<polyline2d>`, `<polyline3d>`, and `<polyline6d>`.

### Rationale

1. **Consistent Naming Across Dimensions:** Explicitly naming `<polyline2d>` and `<point2d>` aligns all three kinematic modes (`2d`, `3d`, `6d`) and removes the assumption that planar 2D is a special-case default.
2. **Closed Contour Support in 3D and 6D:** Non-planar conformal contours and closed multi-axis deposition paths benefit from true closed-loop semantics without repeating the initial vertex (which introduces ambiguity around endpoint dwell/seam behavior). A `closed="true"` attribute enables closed contours uniformly across 2D, 3D, and 6D polylines.
3. **Cleaner Schema & Parser Implementation:**  `<loop>` and `<polyline>` share identical child structures (`<point>` + `<sub>`) and closing-segment semantics (where the first vertex's modifier factors and `tag` govern the implied closing edge back from the final vertex). Merging `<loop>` into `<polyline2d closed="true">` eliminates duplicate complex types in `tp.xsd` and simplifies consumer parsers.

### Proposed Changes

* **Rename Elements (`tp.xsd` & Spec Part II §1.3):**
  * Rename `<polyline>` \$\\rightarrow\$ `<polyline2d>` and `<point>` \$\\rightarrow\$ `<point2d>`.
  * Update `ST_SegmentType` enum value `polyline` \$\\rightarrow\$ `polyline2d` and remove `loop`.
* **Add `closed` Attribute & Remove `<loop>`:**
  * Remove the `<loop>` element.
  * Add `closed` (`xs:boolean`, `use="optional"`, `default="false"`) to `<polyline2d>`, `<polyline3d>`, and `<polyline6d>`.

