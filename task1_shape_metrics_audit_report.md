# Shape Metrics Audit

## Summary Table

| Source | Metrics used | Main use |
|---|---|---|
| `instancemaker` | Area, perimeter, compactness, shape index, interior-edge ratio, fractal dimension | Add polygon-level attributes after map production. These are useful for map audit, rasterized quality layers, and later filtering. |
| Xiong et al. (2026) | Area, compactness, fractal dimension, fractal-dimension residual, corrected field area, estimated field count | Detect under-segmented polygons and adjust field-size estimates without changing the polygon geometry. |
| Rahebe issue #3 | Pixel accuracy, macro IoU, macro precision, macro recall, macro F1; boundary-band IoU, boundary F1, PoLiS, Hausdorff95, tangent-angle disagreement; component-count ratio, area ratio, compactness, shape index, vertex-complexity proxies; object precision, object recall, object F1; Betti-0, Betti-1, Mu0 matching error | Compare predicted and reference chips at pixel, boundary, geometry, object, and topology levels. This is the broadest tested metric set. |
| Fields of the Planet | Object F1, Panoptic Quality, Recognition Quality, Segmentation Quality, matched-boundary chamfer distance, polygon-count error, size-stratified PQ, pixel IoU | Evaluate whether models recover fields as separate parcel objects, especially across image resolutions and field-size classes. |
| FTW Mapping Africa final report | Field area, fractal index | Compare large-scale map outputs from different models and diagnose false positives and poor field separation. |

## Metrics By Group

### 1. Basic Polygon Size

- `area_m2` / `area_ha`
- `perimeter_m`
- field count
- polygon-count error

These are simple and easy to interpret. Area helps identify very small false positives and very large possible merged fields. Field count and polygon-count error help show whether a model is over- or under-producing field objects.

### 2. Shape Regularity

- compactness: `4 * pi * area / perimeter^2`
- shape index: `perimeter / (2 * sqrt(pi * area))`
- interior-edge ratio: `perimeter / area`

Compactness is the clearest regularity metric. Higher compactness usually means cleaner, more regular fields. Low compactness can indicate merged fields, jagged edges, or long narrow shapes. Shape index and interior-edge ratio give similar information in different forms.

### 3. Shape Complexity

- fractal dimension / fractal index: `2 * log(perimeter) / log(area)`
- fractal-dimension residual from the expected area-FD curve
- vertex-complexity proxies

Fractal dimension is already used in multiple project materials. It is useful because under-segmented polygons often have more complex boundaries than regular fields of similar area. In Xiong et al. (2026), the residual from the area-FD relationship is used to estimate how many fields may be hidden inside a merged polygon.

### 4. Boundary Accuracy

- boundary-band IoU
- boundary F1 with tolerance
- PoLiS distance
- Hausdorff95
- tangent-angle disagreement
- matched-boundary chamfer distance

These metrics focus on boundary placement, not only field area. They are useful when the map captures the field extent but places edges poorly, misses narrow boundaries, or draws boundaries with the wrong direction.

### 5. Object And Instance Recovery

- object precision
- object recall
- object F1
- Recognition Quality
- Segmentation Quality
- Panoptic Quality
- size-stratified PQ

These are important because field-boundary maps are used as field objects, not only pixels. Object F1 measures whether the correct fields were recovered. Panoptic Quality combines object recovery and matched-shape quality. Size-stratified PQ is especially useful for smallholder systems because small fields are the easiest to merge or miss.

### 6. Topology

- component-count ratio
- Betti-0 error
- Betti-1 error
- Mu0 matching error

These metrics check object structure. Betti-0 detects split and merge count errors. Betti-1 detects holes. Mu0 matching error checks unmatched predicted and reference components after spatial matching. These are useful diagnostics when pixel scores look acceptable but object structure is wrong.

## Main Findings

1. The same core shape metrics appear across several sources: area, compactness, and fractal dimension.
2. Fractal dimension is the most established metric for under-segmentation in the current project materials.
3. Compactness is the clearest companion metric for detecting merged or irregular fields.
4. Object-level metrics are needed because pixel IoU can hide merged fields, split fields, and false objects.
5. Panoptic Quality is the strongest single evaluation metric from Fields of the Planet because it combines object recovery and shape overlap.
6. Boundary-distance metrics are useful when the field object is detected but the boundary is shifted or poorly localized.
7. Topology metrics are promising diagnostics, but they are less directly interpretable for reporting than area, compactness, fractal dimension, object F1, and PQ.

## Recommended Short Metric Suite For Next Work

For a clear and practical starting suite, use:

| Purpose | Recommended metrics |
|---|---|
| Basic map summary | field count, area, median area |
| False-positive screening | very small field share, area, compactness |
| Under-segmentation flagging | area, compactness, fractal dimension, FD residual |
| Model comparison | object precision, object recall, object F1, Panoptic Quality |
| Small-field performance | size-stratified PQ or size-stratified object F1 |
| Boundary quality | boundary F1 with tolerance, Hausdorff95 or chamfer distance |

This suite is brief but covers the main failure modes: false positives, merged fields, missed fields, split fields, poor geometry, and shifted boundaries.

## Notes For Task 2

The next step should test which metrics are most useful for each map-based application:

- removing false positives
- flagging under-segmentation
- comparing model improvements
- detecting special field types such as center pivots
- evaluating large-scale maps where reference labels are not available

The most likely first candidates are area, compactness, fractal dimension, FD residual, object F1, and Panoptic Quality.
