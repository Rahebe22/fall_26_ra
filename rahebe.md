# Field-Boundary Evaluation Metrics

FTW supervised model run with matched prediction and ground-truth label chips.

## Evaluation Set

The evaluation set contains `3,930` chips. Each chip is `224 x 224` pixels with hard labels:

- `0`: background / non-field
- `1`: field interior
- `2`: field boundary

The main goal is to evaluate field extent, derived boundaries, object geometry, and topology, rather than relying only on pixel accuracy.

The boundary class is used as a diagnostic and as part of the field-label structure.

## Why Pixel Accuracy Is Not Enough

A model can overlap the field area while still drawing poor boundaries, merging adjacent fields, splitting fields into fragments, or creating unrealistic shapes. For this reason, the evaluation uses four groups of metrics.

### Pixel Agreement

Pixel agreement asks whether the model labeled the right pixels as field, background, or boundary.

### Boundary Quality

Boundary quality asks whether predicted field edges are close to and aligned with ground-truth field edges.

### Geometry

Geometry asks whether predicted objects have realistic area, perimeter, compactness, and complexity.

### Topology And Object Structure

Topology and object metrics ask whether the prediction preserves object counts, holes, merges, and splits.

## Pixel-Level Metrics

The main pixel-level metrics keep classes `0`, `1`, and `2` separate. All metrics were calculated on a per-chip basis.

| Metric | Mean | Median |
|---|---:|---:|
| `multiclass_accuracy` | 0.708 | 0.723 |
| `multiclass_macro_iou` | 0.420 | 0.412 |
| `multiclass_macro_precision` | 0.584 | 0.602 |
| `multiclass_macro_recall` | 0.636 | 0.636 |
| `multiclass_macro_f1` | 0.531 | 0.548 |

### `multiclass_accuracy`

Multiclass accuracy is the fraction of pixels where the predicted hard label exactly equals the ground-truth hard label.

### `multiclass_macro_iou`

Macro IoU calculates IoU separately for classes `0`, `1`, and `2`, then averages the class scores. This keeps the field interior and boundary classes separate.

### `multiclass_macro_precision`

Macro precision calculates precision separately for classes `0`, `1`, and `2`, then averages the class scores.

### `multiclass_macro_recall`

Macro recall calculates recall separately for classes `0`, `1`, and `2`, then averages the class scores.

### `multiclass_macro_f1`

Macro F1 calculates F1 separately for classes `0`, `1`, and `2`, then averages the class scores.

### Extent Diagnostics

The `extent_*` metrics are still exported, but they collapse labels `1` and `2` into one field-related class. They should be interpreted only as field-presence diagnostics, not exact three-class pixel agreement.

## Class-Wise Pixel-Level Metrics

Class-wise pixel metrics are summarized as the mean of per-chip class-wise metrics.

<img width="1200" height="760" alt="Image" src="https://github.com/user-attachments/assets/8d3a5193-0afd-4129-bba1-bea6021cdd91" />

### Class `0`: Background

Background performance helps diagnose false positives, but it can dominate the chip.

### Class `1`: Field Interior

Field interior is the basis for connected components, geometry, and topology metrics.

### Class `2`: Boundary

The boundary class is evaluated as a diagnostic, not as the primary target.

## Boundary-Quality Metrics

The main boundary-quality metrics are derived from class-`1` field-interior objects.

First, connected components are extracted from class-`1` field interiors. Then the boundary of each field-interior object is derived. Finally, predicted and ground-truth object boundaries are compared using tolerance, distance, and orientation metrics.

Object, geometry, and topology metrics are still extracted from class-`1` interiors so adjacent fields are not merged by the boundary pixels.

| Metric | Mean | Median |
|---|---:|---:|
| `object_boundary_band_iou` | 0.264 | 0.288 |
| `boundary_f1_tol` | 0.452 | 0.486 |
| `polis_px` | 8.548 | 4.679 |
| `hausdorff95_px` | 36.632 | 20.242 |
| `mta_mean_deg` | 17.820 | 18.635 |

### `object_boundary_band_iou`

Object boundary-band IoU compares dilated object-derived boundaries. It allows small shifts instead of requiring exact pixel overlap.

### `boundary_f1_tol`

Boundary F1 with tolerance is the F1 score for boundary points within the pixel tolerance, which is `2` pixels in this calculation. It separates near-misses from completely missed edges.

### `polis_px`

PoLiS distance is the average symmetric nearest-boundary distance in pixels. Lower values mean boundaries are closer to the reference.

### `hausdorff95_px`

Hausdorff95 is the 95th-percentile boundary distance. It captures large local failures without using the single worst pixel.

### `mta_mean_deg`

Mean tangent-angle disagreement measures whether nearby boundaries point in the same direction.

## Pixel And Boundary Errors

<img width="1172" height="409" alt="Image" src="https://github.com/user-attachments/assets/ff320ed6-a0c5-4561-b61c-577ff383059e" />

This figure shows one matched chip example using ground-truth labels, predicted labels, field-extent errors, and object-boundary overlap.

## Boundary Tolerance Bands

<img width="897" height="407" alt="Image" src="https://github.com/user-attachments/assets/b6960f76-eb2c-4d2d-9078-2b5d4a6c51eb" />

Thin outlines show predicted and ground-truth object-derived edges. The two-pixel band dilates each edge so small shifts count as partial agreement. Bright pixels in the disagreement panel mark remaining mismatches.

## Boundary Metric Distributions

<img width="652" height="728" alt="Image" src="https://github.com/user-attachments/assets/c313b069-6ddc-4411-b219-cb1200f79ba2" />

Boundary-band IoU shows boundary-band overlap, PoLiS exposes boundary shifts, tolerance F1 captures nearby edges, and tangent-angle disagreement checks boundary direction.

## Geometry Metrics

Geometry is computed from connected components in the class-`1` field-interior mask.

| Metric | Mean | Median |
|---|---:|---:|
| `component_count_ratio` | 1.237 | 1.000 |
| `area_ratio` | 1.922 | 0.918 |
| `pred_mean_compactness` | 0.440 | 0.442 |
| `pred_median_shape_index` | 1.508 | 1.475 |
| `n_ratio_vertex_proxy` | 2.212 | 1.583 |
| `c_iou_vertex_proxy` | 0.419 | 0.444 |

### `component_count_ratio`

Component-count ratio is the predicted component count divided by the ground-truth component count. Values above `1` suggest over-fragmentation, while values below `1` suggest missing or merged fields.

### `area_ratio`

Area ratio is the predicted interior area divided by the ground-truth interior area. It shows over- or under-mapping of field interiors.

### Compactness

Compactness describes shape regularity from area and perimeter. Higher values mean more compact objects, while lower values mean elongated or jagged objects.

### Shape Index

Shape index is a shape-complexity measure. A value of `1` is very compact; larger values mean more irregular or elongated shapes.

### `n_ratio_vertex_proxy`

This is a raster proxy for boundary or shape complexity. Values above `1` mean predicted objects are more complex than ground-truth objects.

### `c_iou_vertex_proxy`

This is an overlap score adjusted by shape complexity, so it is stricter than plain extent IoU.

## Geometry Distributions

<img width="1460" height="1604" alt="Image" src="https://github.com/user-attachments/assets/bd1641f2-99be-406f-900b-2b199abcb2e9" />

Count ratio shows merge or split tendency. Area ratio shows over- or under-mapping of field interiors. Complexity proxies show whether shapes are too simple or jagged.

## Topology And Object Metrics

These metrics evaluate splits, merges, holes, and object correspondence.

| Metric | Mean | Median |
|---|---:|---:|
| `object_precision` | 0.569 | 0.593 |
| `object_recall` | 0.558 | 0.583 |
| `object_f1` | 0.512 | 0.549 |
| `betti0_abs_error` | 10.070 | 6.000 |
| `betti1_abs_error` | 2.812 | 1.000 |
| `mu0_matching_error` | 17.558 | 14.000 |

### `object_precision`

Object precision is matched predicted components divided by predicted components. Low values mean false objects or badly placed fragments.

### `object_recall`

Object recall is matched ground-truth components divided by ground-truth components. Low values mean missing fields or severe merges and splits.

### `object_f1`

Object F1 is the balanced object-level score. It is the closest metric in this group to asking whether the model recovered the field instances.

### `betti0_abs_error`

Betti-0 absolute error is the absolute error in connected-component count. It detects split and merge count errors. Lower is better.

### `betti1_abs_error`

Betti-1 absolute error is the absolute error in hole or cycle count. It detects artificial or missing holes. Lower is better.

### `mu0_matching_error`

Mu0 matching error is unmatched predicted components plus unmatched ground-truth components after spatial matching. It checks object correspondence, not only counts. Lower is better.

## Worst Topology Case

<img width="1180" height="444" alt="Image" src="https://github.com/user-attachments/assets/6a27bc84-4c75-4303-9c1b-63749ec421d5" />

Chip `2176` has `mu0_matching_error = 168` in the matched run. This means many predicted and ground-truth connected components cannot be paired spatially.

This is why object and topology metrics are needed: the field area can still overlap while individual fields are merged, missing, or fragmented.

## Additional High Topology-Error Examples

<img width="1167" height="453" alt="Image" src="https://github.com/user-attachments/assets/4f16f191-3c6e-474b-85dc-298dcd56d263" />

Chip `559` has `mu0_matching_error = 158`.

<img width="1165" height="457" alt="Image" src="https://github.com/user-attachments/assets/7ace9974-16da-45ba-a7f0-ab9c8cc994c2" />

Chip `953` has `mu0_matching_error = 129`.

These examples show strong object-structure disagreement. Ground truth and prediction contain different connected field systems, even where some area overlap exists.

## Chip Examples

The chip examples make the metrics interpretable.

In the field-extent panels, green is correct field extent, orange is prediction only, and blue is ground truth only. In the boundary panels, purple means shared boundary.

### No Predicted Extent

<img width="1260" height="454" alt="Image" src="https://github.com/user-attachments/assets/702b3a31-8920-4598-8e1a-504b5a4635b0" />

The prediction is all background while the ground truth contains fields.

### No Ground-Truth Extent

<img width="1260" height="454" alt="Image" src="https://github.com/user-attachments/assets/ad0eea27-c369-415e-aa79-e6e1dd6b2061" />

The ground truth is all background while the model predicts fields.

### Zero Object F1

<img width="1260" height="454" alt="Image" src="https://github.com/user-attachments/assets/5e64b328-09c2-45ef-892c-1b30d336b0b7" />

Field extent can overlap while object structure is wrong.

## Recommended Model-Comparison Table

| Category | Metric 1 | Metric 2 | Meaning |
|---|---|---|---|
| Pixel agreement | `multiclass_macro_iou` | `multiclass_macro_f1` | Exact three-class pixel agreement. |
| Boundary quality | `boundary_f1_tol` | `polis_px` | Spatial alignment of derived boundaries. |
| Geometry | `area_ratio` | `component_count_ratio` | Object size and count behavior. |
| Topology/object | `object_f1` | `mu0_matching_error` | Field instance correspondence. |

The matched filename table should be kept with every run because it is the audit trail that makes the metric comparison reproducible.
