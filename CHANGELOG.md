# cavalier_contours changelog

All notable changes to the cavalier_contours crate will be documented in this file.

## Unreleased

### Added ⭐

- Added `BooleanResultInfo` enum used for new `result_info` field on `BooleanResult`. The enum
  is used to return information about what happened during the boolean operation.
- Added doc tests and improved documentation to intersect functions `circle_circle_intr`,
  `line_circle_intr`, and `line_line_intr`
  ([#18](https://github.com/jbuckmccready/cavalier_contours/pull/18)).

### Changed 🔧

- Added epsilon parameter to `PlineSource::closest_point` method to allow for consistency with other
  calculations using epsilon values for fuzzy comparing.
- `line_line_intr` function now scales parametric t values by segment lengths before fuzzy comparing
  with epsilon value to avoid introduced error when line segments are very long or very short.
- `pline_seg_intr` function now scales parametric t values by segment lengths and angle values by
  arc radii before fuzzy comparing with epsilon value to avoid introduced error when line segments
  are very long or very short, or arc radius is very small or very large.
- Reimplemented `line_circle_intr` function for improved numerical stability.
- Added epsilon parameter to `parametric_from_point` function to allow consistency with other
  calculations using epsilon values for fuzzy comparing.
- Reduce error in calculation done in `parametric_from_point` by using larger component difference
  rather than explicitly checking for vertical line case.
- Added epsilon parameter to `point_within_arc_sweep` function to allow for consistency with other
  calculations using epsilon values for fuzzy comparing.
- Added `#[inline]` attribute to all of the small Vector2 and base math functions.

### Fixed 🐛

- Fixed `PlineSource::remove_redundant` to use epsilon values in all fuzzy compares. Previously
  used default epsilon value from Num trait in some comparisons. If epsilon value passed in is less
  than default Num trait value (`1e-8` for `f32` and `f64` values) then it is more aggressive in
  removing vertexes.
- Fixed pline `find_intersects` function to use epsilon parameter in all cases.
- Fixed boolean operation bugs caused by `line_circle_intr` numerical stability problem,
  inconsistencies between epsilon values used across functions, and lack of scaling parametric t
  values and angles for fuzzy comparing with epsilon values
  ([#23](https://github.com/jbuckmccready/cavalier_contours/issues/23)).
- Fixed some of the doc comments around pline boolean operation types.
- Fixed polyline find_intersects to use pos_equal_eps passed in options for querying bounding boxes
  ([#22](https://github.com/jbuckmccready/cavalier_contours/pull/22)).
- Fixed `PlineViewData::from_new_start` to not discard bulge value if start point lies on top of the
  first vertex of a closed polyline, and added tests to cover case.

## 0.2.0 - 2022-05-12

### Added ⭐

- Added CHANGELOG.md file for tracking changes and releases.
- New traits `PlineSource`, `PlineSourceMut`, and `PlineCreation` for sharing methods across
  different polyline data views (for example sub views/selections over polylines or direction
  inversion).
- `PlineViewData` and `PlineView` types. `PlineView` implements `PlineSource` trait allowing for
  flexible views over polyline data to be processed with all the shared behavior/methods.
  `PlineViewData` holds the data used to index into an underlying source polyline to form a
  `PlineView`.

### Changed 🔧

- All Polyline methods have moved to the appropriate trait (`PlineSource`, `PlineSourceMut`, or
  `PlineCreation`).
- Slices used in offset and boolean operations now all use new `PlineViewData` and `PlineView`
  types.
- Changed polyline methods that returned `Cow` types to return `Option` types to indicate if changes
  were done on the input polyline or not.
- Changed intersect functions (`circle_circle_intr`, `line_circle_intr`, and `line_line_intr`) to
  accept epsilon value parameter for fuzzy float comparisons.
- Changed polyline offset and boolean operations to use `pos_equal_eps` epsilon value in intersect
  functions.
- Changed `pline_seg_intr` function to avoid inconsistencies between intersect functions in some
  cases involving line-arc intersects due to fuzzy comparing - this fixes problems that can arise in
  operations that depend on finding intersects.
- Added `include_overlapping` parameter to `all_self_intersects_as_basic` function.

### Fixed 🐛

- Fixed arc overlap intersects not being found in some cases and segment intersects disagreeing due
  to fuzzy comparing causing incorrect output for polyline boolean operations and added test cases
  that found the issue.
- Fixed bug in parallel offset caused when raw offset polyline produced an overlapping self
  intersect. Parallel offset algorithm no longer includes overlapping intersects when finding self
  intersects of raw offset polyline for creating slices.

### Removed 🔥

- `Polyline::visit_segments` (use `PlineSource::iter_segments` instead).
- `Polyline::len` method removed (use `PlineSource::vertex_count` instead).
- `PolylineSlice` trait and `OpenPlineSlice` type (covered by new view types and polyline traits).
- `PolylineSlice::stitch_onto` and `PolylineSlice::to_polyline` removed, to get same functionality
  construct the view with `PlineData::view` and pass it to `PlineSourceMut::extend_remove_repeat` or
  `PlineCreation::create_from_remove_repeat`.
