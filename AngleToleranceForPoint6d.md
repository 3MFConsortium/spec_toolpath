**Title:** Add optional orientation tolerance attribute to `<point6d>` to support kinematic deadband optimization in multi-axis IK solvers

### Summary

Currently, `<point6d>` defines an exact unit quaternion $(i, j, k, w)$ at each waypoint, requiring inverse kinematics (IK) solvers to treat every target tool orientation as a strict constraint. In multi-axis additive manufacturing (e.g., 5-axis tilting/rotating beds or 6-axis robotic deposition), enforcing exact orientation on waypoints with minor surface curvature causes unnecessary rotary axis micro-adjustments, slowing build times and introducing surface artifacts.

This proposal adds an optional angular tolerance attribute (e.g., `angletolerance`, defaulting to `0`) to `<point6d>` so slicers and toolpath planners can communicate allowable orientation slack to downstream machine controllers and IK solvers.

---

### Motivation & Use Case

1. **Avoiding Rotary Axis Thrashing on Gentle Curvature:** When printing conformal layers or non-planar trajectories over shallow curves, the tool orientation change of only a few degrees often has negligible impact on bead quality or overhang support. However, if a strict nozzle direction is required,  then there may be significant physical travel on slow rotary axes (tilt rotations, turntable rotations or robotic wrist joints).
2. **Deadband / Hysteresis Optimization in IK Solvers:** When an angular tolerance $\theta_{\text{tol}}$ is provided, an IK solver can check whether the target tool axis $\hat{z}'_{\text{target}}$ (which is often the normal of printing layer) falls within the tolerance cone of the machine's realized tool orientation $\hat{z}'_{\text{realized}}$:
   $$
   \hat{z}'_{\text{target}} \cdot \hat{z}'_{\text{realized}} \ge \cos(\theta_{\text{tol}})
   $$

If satisfied, the solver can hold the current rotary joint angles steady and execute pure linear $(X, Y, Z)$ translation, dramatically improving motion smoothness and cycle time.

### Proposed Changes

* **XSD Schema (`tp.xsd`):** Add an optional non-negative attribute (e.g., `angletolerance` in degrees or radians, default `0.0`) to `<point6d>`.
* **Specification Text (Part II §1.4.2 – Polyline Segment, 6-axis):**
  * Define `angletolerance` as the maximum allowable angular deviation between the realized tool axis $\hat{z}'$ and the target orientation specified by $(i, j, k, w)$.
  * Specify that a value of `0` (or omission of the attribute) requires exact orientation tracking, and that consumers without IK relaxation support MAY safely ignore the attribute.

